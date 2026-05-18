# Trade Busts

Reya is a hybrid system where the matching engine runs off-chain and the chain independently validates and settles each trade. If a trade that the matching engine matched fails on-chain validation, it results in a **trade bust** — the match is rolled back, both sides are released back to their previous state, and no trade occurred.

Busts are an expected but **uncommon** outcome of this architecture. The Reya team continuously works to minimize the edge cases that produce them — by tightening pre-match validation in the matching engine, narrowing the time window between match and settle, and reducing the conditions under which on-chain settlement can fail. Busts will never be zero in a hybrid system (the chain is always the ultimate source of truth), but they are designed to be rare given how disruptive they are to client workflows.

This page explains why busts happen, when in the trade lifecycle they happen, and how a client should handle them.

## The Two-Stage Trade Lifecycle

Every match on Reya goes through **two distinct stages**:

1. **Off-chain match.** The matching engine (ME) finds a counterparty and produces a fill record. This is fast and in-memory.
2. **On-chain settlement.** The executor submits the fill to the `OrdersGateway` smart contract. Each fill is validated on-chain (signatures, balances, margin requirements, oracle deviation, etc.) before being accepted.

These stages have independent outcomes. **A successful off-chain match does not guarantee on-chain settlement.** When settlement fails, the result is a bust.

Different WebSocket channels surface different stages of this lifecycle, which is what makes bust handling subtle — but also what makes it manageable, once you know which channel reflects which stage.

## Why Busts Happen

The matching engine and the on-chain settlement layer operate on slightly different snapshots of the world. The matching engine matches based on its in-memory view of orders, balances, and prices at the moment of the match. The on-chain settlement contract re-validates every fill against the **current** chain state when the settlement transaction lands — which may be a fraction of a second later, but enough time for some things to have changed.

The most common reasons settlement fails:

- **Balance or margin moved.** A previous fill on the same account consumed collateral, or a position-size change pushed the account toward (or past) its margin requirements. The match was viable when the ME saw it; by the time the contract checked, it no longer was.
- **Oracle price guard.** Each order signature commits to a price band (e.g. an IOC order's worst acceptable price). If the on-chain oracle has moved outside that band by settlement time, the contract refuses the fill rather than trade at an unacceptable price.
- **Order expired.** Orders carry an `expiresAfter` deadline as part of their signature. If on-chain settlement runs after that deadline (e.g. because of network congestion or executor backlog), the contract refuses the fill.
- **Signer permissions changed.** The signing wallet's on-chain permissions (e.g. removal from an account's authorized signers, or a scope change) were revoked between signing and settlement.
- **Market state changed.** The market was paused, delisted, or had its parameters changed on-chain between match and settle.
- **Nonce already used.** Rare — typically indicates a client-side bug retrying the same signed order rather than re-signing with a fresh nonce.

All of these share a common pattern: the matching engine and the chain disagreed about the state of the world at slightly different points in time, and the chain — being the source of truth — wins the disagreement. The team's work to minimize busts focuses on shortening the time window where this disagreement can develop, and on catching more of the conditions in pre-match validation so the ME never matches a trade that the chain would later reject.

## Timeline of a Trade

End-to-end for a single fill, in order:

1. A client submits an order via `POST /v2/createOrder` (or the equivalent WebSocket Order Entry call).
2. The matching engine finds a counterparty and generates the fill.
3. The ME publishes the order status change to Redis. This feeds the [`/v2/wallet/{address}/orderChanges`](websocket-api-reference.md) WebSocket channel. At this point the maker side of the match surfaces — as `FILLED` if the maker order was fully consumed, or as `OPEN` with an updated cumulative-fill quantity if it was only partially consumed. **This is before on-chain settlement.**
4. The executor submits the fill to the `OrdersGateway` smart contract on Reya Network.
5. The contract validates the fill on-chain independently of the match — re-checking signatures, balances, margin, oracle staleness, market state, and so on.
6. The settlement outcome is published:
   - **Success.** The contract emits a successful-settlement event, which is indexed and published to [`/v2/wallet/{address}/spotExecutions`](websocket-api-reference.md) (and the corresponding `/v2/market/{symbol}/spotExecutions` channel). This is the confirmed trade.
   - **Failure (bust).** The contract emits a settlement-failure event, which is indexed and published to [`/v2/wallet/{address}/spotExecutionBusts`](websocket-api-reference.md) (and the corresponding `/v2/market/{symbol}/spotExecutionBusts` channel). The trade did not happen; both sides are released.

## The Three Channels — What Each One Tells You

| Channel | What it shows | Reflects on-chain settlement? |
|---|---|---|
| `/v2/wallet/{address}/orderChanges` | ME-level order status changes (`OPEN`, `FILLED`, `CANCELLED`, `REJECTED`), plus updated cumulative-fill quantity | **No** — fires immediately after the ME matches, before settlement |
| `/v2/wallet/{address}/spotExecutions` | Confirmed on-chain trades | **Yes** — only emitted after settlement succeeds |
| `/v2/wallet/{address}/spotExecutionBusts` | Failed on-chain settlements | **Yes** — only emitted after settlement fails |

The critical asymmetry: **`orderChanges` reports ME-level state and never retroactively corrects itself.** If a fill briefly appeared as `FILLED` on `orderChanges` and then the on-chain settlement attempt busted, no correction event is published on `orderChanges`. The order's state on that channel will reflect what the ME observed, not what happened on-chain.

## Recommended Pattern

For most clients, the cleanest model is to treat **`spotExecutions` as the single source of truth for confirmed trades.**

- If a trade appears on `spotExecutions`, it has settled on-chain and is final. It can never be busted after that.
- If a trade does not appear, either it never matched, or it matched but busted on settlement. Either way, your position state is unaffected and there is nothing to reconcile.

Under this model:

- **`spotExecutions`** drives your trade ledger and position accounting.
- **`orderChanges`** is useful for surfacing intent and ME-level workflow (e.g. "my order is now resting", "my order has been cancelled"), but its `FILLED` status (or any non-zero cumulative-fill quantity on an `OPEN` order) is not a trade confirmation — those signals fire before on-chain settlement.
- **`spotExecutionBusts`** is optional. You don't need to subscribe to it to be correct — busted trades simply never appear on `spotExecutions` in the first place. Subscribe only if you specifically need awareness of failed settlements, e.g. for monitoring, alerting on systematic failure rates, or surfacing diagnostic information to end users.

## What Not to Do

- **Don't treat the order-submission response as a trade confirmation.** The response from `POST /v2/createOrder` (REST) or `createOrder` over WebSocket Order Entry reports what the matching engine did with your order — it carries no information about whether any resulting fill settled on-chain. Treat it the same way you'd treat an `orderChanges` update. See the dedicated section below.
- **Don't treat `FILLED` on `orderChanges` as a trade confirmation.** It is an ME-level status that fires before on-chain settlement. A fill seen here may still be busted later, and you will not be notified of the bust on this channel.
- **Don't try to "match" `orderChanges` events against `spotExecutions` to detect busts.** It's possible to do, but it's strictly harder than just listening to `spotExecutionBusts` directly (or ignoring busts entirely and relying on `spotExecutions` as the source of truth).
- **Don't double-count.** A trade that settles on-chain produces exactly one event on `spotExecutions`. A trade that busts produces exactly one event on `spotExecutionBusts`. Never both.

## The Order-Submission Response Is an ME-Level Signal

When you submit an order — whether over REST (`POST /v2/createOrder`, `POST /v2/cancelOrder`, `POST /v2/cancelAll`) or the equivalent WebSocket Order Entry operations — the response you get back reports what the **matching engine** did with your order. It does **not** report on-chain settlement.

Treat the order-submission response the same way you'd treat an update on the `orderChanges` channel:

- **Success response with fills in the body.** The matching engine matched your order and produced fill records. Those fills are ME-level fills; they have not yet been settled on-chain and may still bust during settlement. The presence of fills in the response is not a guarantee that any of them settle.
- **Success response without fills.** The matching engine accepted your order — it rested on the book (GTC) or didn't find a match (IOC). No settlement is implied either way.
- **Failure response.** The matching engine rejected the order before any matching attempt (validation failure, signature error, insufficient permissions, rate-limit, etc.). No matches were attempted, so no fills exist anywhere.

In none of these cases does the response carry settlement information. To learn whether a specific fill settled, listen to [`/v2/wallet/{address}/spotExecutions`](websocket-api-reference.md). That channel is the only authoritative source for confirmed trades — if a fill appears there, it has settled on-chain and is final; if it never appears, either it never matched or it busted.

This rule is the same regardless of which transport you use to submit the order. REST and WebSocket Order Entry affect latency and framing; they don't change the finality semantics. **Finality is on-chain, and finality is reported only on `spotExecutions`.**

The same applies in the partial-fill case. If you submit an aggressive order that produces ten fills, you will see references to those fills in the submission response (and on `orderChanges`), but each of the ten is independently subject to on-chain settlement. Some may settle, some may bust. The submission response and `orderChanges` will not reflect which is which — only `spotExecutions` (for the ones that settled) and `spotExecutionBusts` (for the ones that busted) will.

## Reading a Bust Event

A bust event on `spotExecutionBusts` carries enough information to identify which match failed and why. The wire shape and field semantics are documented in the channel reference under [`/v2/market/{symbol}/spotExecutionBusts`](websocket-api-reference.md). Key fields:

- `symbol`, `accountId`, `makerAccountId`, `orderId`, `makerOrderId`, `qty`, `side`, `price` — identify the failed match
- `reason` — the hex-encoded revert reason bytes from the on-chain settlement attempt. Clients can ABI-decode this against the `OrdersGateway` error ABI to recover the specific revert (e.g. an `InsufficientBalance` or `UnauthorizedSigner` error).

For most monitoring needs, the high-level information (which order, which counterparty, why) is sufficient. Decoding the `reason` bytes is only necessary if you want to drive automated remediation based on the specific failure mode.
