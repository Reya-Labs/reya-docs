# Trade Busts

A **trade bust** is what happens when the matching engine produces a fill, but the resulting on-chain settlement attempt fails. The match is rolled back; both sides are released back to their previous state as if the fill had never occurred.

This page explains why busts can happen, when they happen in the trade lifecycle, and how a client should handle them.

## The Two-Stage Trade Lifecycle

Every match on Reya goes through **two distinct stages**:

1. **Off-chain match.** The matching engine (ME) finds a counterparty and produces a fill record. This is fast and in-memory.
2. **On-chain settlement.** The executor submits the fill to the `OrdersGateway` smart contract. Each fill is validated on-chain (signatures, balances, margin requirements, oracle deviation, etc.) before being accepted.

These stages have independent outcomes. **A successful off-chain match does not guarantee on-chain settlement.** When settlement fails, the result is a bust.

Different WebSocket channels surface different stages of this lifecycle, which is what makes bust handling subtle — but also what makes it manageable, once you know which channel reflects which stage.

## Timeline of a Trade

End-to-end for a single fill, in order:

1. A client submits an order via `POST /v2/createOrder` (or the equivalent WebSocket Order Entry call).
2. The matching engine finds a counterparty and generates the fill.
3. The ME publishes the order status change to Redis. This feeds the [`/v2/wallet/{address}/orderChanges`](websocket-api-reference.md) WebSocket channel. At this point, a resting maker order may already show as `PARTIALLY_FILLED` or `FILLED`. **This is before on-chain settlement.**
4. The executor submits the fill to the `OrdersGateway` smart contract on Reya Network.
5. The contract validates the fill on-chain independently of the match — re-checking signatures, balances, margin, oracle staleness, market state, and so on.
6. The settlement outcome is published:
   - **Success.** The contract emits a successful-settlement event, which is indexed and published to [`/v2/wallet/{address}/spotExecutions`](websocket-api-reference.md) (and the corresponding `/v2/market/{symbol}/spotExecutions` channel). This is the confirmed trade.
   - **Failure (bust).** The contract emits a settlement-failure event, which is indexed and published to [`/v2/wallet/{address}/spotExecutionBusts`](websocket-api-reference.md) (and the corresponding `/v2/market/{symbol}/spotExecutionBusts` channel). The trade did not happen; both sides are released.

## The Three Channels — What Each One Tells You

| Channel | What it shows | Reflects on-chain settlement? |
|---|---|---|
| `/v2/wallet/{address}/orderChanges` | ME-level order status changes (`OPEN`, `PARTIALLY_FILLED`, `FILLED`, `CANCELLED`) | **No** — fires immediately after the ME matches, before settlement |
| `/v2/wallet/{address}/spotExecutions` | Confirmed on-chain trades | **Yes** — only emitted after settlement succeeds |
| `/v2/wallet/{address}/spotExecutionBusts` | Failed on-chain settlements | **Yes** — only emitted after settlement fails |

The critical asymmetry: **`orderChanges` reports ME-level state and never retroactively corrects itself.** If a fill briefly appeared as `FILLED` on `orderChanges` and then the on-chain settlement attempt busted, no correction event is published on `orderChanges`. The order's state on that channel will reflect what the ME observed, not what happened on-chain.

## Recommended Pattern

For most clients, the cleanest model is to treat **`spotExecutions` as the single source of truth for confirmed trades.**

- If a trade appears on `spotExecutions`, it has settled on-chain and is final. It can never be busted after that.
- If a trade does not appear, either it never matched, or it matched but busted on settlement. Either way, your position state is unaffected and there is nothing to reconcile.

Under this model:

- **`spotExecutions`** drives your trade ledger and position accounting.
- **`orderChanges`** is useful for surfacing intent and ME-level workflow (e.g. "my order is now resting", "my order has been cancelled"), but its `FILLED`/`PARTIALLY_FILLED` statuses are not trade confirmations.
- **`spotExecutionBusts`** is optional. You don't need to subscribe to it to be correct — busted trades simply never appear on `spotExecutions` in the first place. Subscribe only if you specifically need awareness of failed settlements, e.g. for monitoring, alerting on systematic failure rates, or surfacing diagnostic information to end users.

## What Not to Do

- **Don't treat `FILLED` on `orderChanges` as a trade confirmation.** It is an ME-level status that fires before on-chain settlement. A fill seen here may still be busted later, and you will not be notified of the bust on this channel.
- **Don't try to "match" `orderChanges` events against `spotExecutions` to detect busts.** It's possible to do, but it's strictly harder than just listening to `spotExecutionBusts` directly (or ignoring busts entirely and relying on `spotExecutions` as the source of truth).
- **Don't double-count.** A trade that settles on-chain produces exactly one event on `spotExecutions`. A trade that busts produces exactly one event on `spotExecutionBusts`. Never both.

## Relationship to the HTTP `createOrder` Response

When you submit an order via `POST /v2/createOrder`, the HTTP request **waits for on-chain settlement to complete before returning the response.** So:

- A success response with fills in the body indicates the fill settled successfully on-chain. No subsequent bust is possible for that fill.
- An error response indicates either the order was rejected before matching, or a match was attempted but settlement failed on-chain (a bust). In either case, no trade occurred and your state is unaffected.

WebSocket listeners on `orderChanges` may see status transitions *before* the HTTP response returns, because `orderChanges` publishes at ME-match time while the HTTP caller is still waiting for settlement. The HTTP response is authoritative; the intermediate `orderChanges` event is not.

The same authoritative-finality property applies to fills emitted on `spotExecutions` and to bust events on `spotExecutionBusts` — both fire only after the on-chain settlement outcome is known.

## Reading a Bust Event

A bust event on `spotExecutionBusts` carries enough information to identify which match failed and why. The wire shape and field semantics are documented in the channel reference under [`/v2/market/{symbol}/spotExecutionBusts`](websocket-api-reference.md). Key fields:

- `symbol`, `accountId`, `makerAccountId`, `orderId`, `makerOrderId`, `qty`, `side`, `price` — identify the failed match
- `reason` — the hex-encoded revert reason bytes from the on-chain settlement attempt. Clients can ABI-decode this against the `OrdersGateway` error ABI to recover the specific revert (e.g. an `InsufficientBalance` or `UnauthorizedSigner` error).

For most monitoring needs, the high-level information (which order, which counterparty, why) is sufficient. Decoding the `reason` bytes is only necessary if you want to drive automated remediation based on the specific failure mode.
