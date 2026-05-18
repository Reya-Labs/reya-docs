# Rate Limits

Reya enforces rate limits at two layers. At the **application layer**, per-wallet limits cap how many trading operations a signing wallet can submit and how many resting orders it can keep on the book — these keep the matching engine and on-chain settlement layer responsive for everyone. At the **network edge**, a separate per-IP / per-API-key limit caps total REST traffic regardless of endpoint, applied at the infrastructure layer before traffic reaches the application. A request has to pass both layers to succeed. This page covers both, what the defaults are, and how to integrate cleanly against them.

The per-wallet limits apply uniformly across both transports — REST (`POST /v2/createOrder`, `POST /v2/cancelOrder`, `POST /v2/cancelAll`) and the corresponding WebSocket Order Entry messages. Switching transports does not bypass them; both feed the same per-wallet bucket. The network-edge limit applies to REST only, since it counts individual HTTP requests.

## How Rate Limits Are Applied

A few properties are important to understand before reading the numbers:

- **Per signer wallet.** Limits are keyed off the wallet that signs the request, not the source IP, not an API key, and not the `accountId`. All requests signed by the same wallet share one bucket regardless of which client or machine issued them.
- **Same bucket across REST and WebSocket.** Sending the same operation over the WebSocket Order Entry API counts the same as sending it over REST. The transport is irrelevant; the signer wallet is what matters.
- **Two distinct limit systems run side by side.** A request must clear **both** to succeed:
  1. **Request-rate limits** — a sliding window measuring how many operations you've sent in the last 60 seconds.
  2. **Open-order caps** — a state-based check on how many resting orders you currently have on the book, and their total notional value.

A request that's fine on one system but trips the other is rejected. You'll usually see whichever limit was hit named in the error message.

## The Three Tiers

Every wallet sits in one of three tiers. The tier determines both your request-rate limits and your open-order caps.

- **Tier 1 (default).** Every wallet starts here. No allowlisting needed.
- **Tier 2 (elevated).** Higher request-rate limits and higher open-order caps.
- **Tier 3 (highest).** Highest request-rate limits and no open-order caps.

**How to move tiers:** contact the Reya team to request an upgrade. The team reviews based on your integration profile, usage patterns, and business context. There is no self-service path to a higher tier.

Only Tier 1 limits are published below. Tier 2 and Tier 3 limits are higher, but the specifics are discussed case-by-case when a wallet is upgraded — the right limit depends on the integration. If you're hitting Tier 1 caps and your use case justifies more, talk to the team rather than designing around them.

## Request Rate Limits (Tier 1)

| Operation | Limit per 60 seconds |
|---|---|
| Order placement and single cancellation (`createOrder` + `cancelOrder`) | **30** |
| Mass cancellation (`cancelAll`) | **10** |

The two counters are independent — placements and single cancels share one bucket of 30; mass-cancels have their own bucket of 10.

**Algorithm: sliding window.** Each request is timestamped. The server counts how many of your requests have arrived in the **last 60 seconds** when a new request comes in. The window slides continuously — a request from 61 seconds ago has already aged out, so a steady cadence of one request per 2 seconds will never trip the limit.

This is different from a fixed-window rate limit. With a fixed window, a "30 per minute" limit would let you send 30 requests in the last second of one minute and another 30 in the first second of the next — 60 requests in 2 seconds. The sliding window prevents this burst.

## Open-Order Caps (Tier 1)

Independent of the request-rate limits, your wallet has a cap on how many open orders can be resting on the book at the same time, and on the total notional value of those orders.

| Limit | Tier 1 |
|---|---|
| Maximum open GTC orders | **100** |
| Maximum total notional of open GTC orders | **5,000 USD** |

These apply to GTC (Good-Till-Cancelled) orders that are sitting in the order book waiting to match. IOC (Immediate-or-Cancel) orders do not count — they either fill at request time or are cancelled, so they never rest on the book.

The cap is checked at order placement. If a new GTC order would push either the count or the notional total over the limit, the placement is rejected. You can free up space under the cap by waiting for fills, cancelling existing orders, or letting orders expire.

## Network-Edge Rate Limit (Per IP / API Key)

Separate from the per-wallet limits above, Reya operates an additional rate limiter at the **network edge** — at the infrastructure layer, before traffic reaches the application. The edge limit is keyed off the source **IP address / API key** rather than the signing wallet, and it applies to **all REST endpoints collectively**, not just trading operations.

| Limit | Value |
|---|---|
| Requests per 60-second window | **1,000** |
| Scope | Per IP / per API key, across all REST endpoints |

A few properties to know:

- **Fixed window.** The counter resets at the end of each 60-second window. You can consume the entire quota at any point inside the window — for example, all 1,000 requests in the first second — and additional requests are rejected until the window resets. This is different from the per-wallet request-rate limit above, which uses a sliding window.
- **All endpoints share one bucket.** Trading requests count against the same 1,000 as any other REST traffic you send (account lookups, market data polls, anything else). There is no per-endpoint sub-allocation.
- **REST only.** The edge limit counts HTTP requests. Messages sent over an established WebSocket connection don't tick the counter — so WebSocket Info subscriptions and WebSocket Order Entry operations sidestep this limit entirely.
- **Independent of the per-wallet limits.** A request must clear both the edge limit and the per-wallet limits to succeed. Upgrading your wallet's tier does not raise the edge limit, and vice versa.
- **Rejection.** Once the limit is reached, additional requests in the same window receive HTTP `429 Too Many Requests` until the window resets.

**Polling vs. WebSocket.** The edge limit is the strongest reason to prefer the [WebSocket Info API](websocket-api-reference.md) over polling REST for any real-time data need. A polling client at one request per second already uses 60 of the 1,000 per minute on a single endpoint — before any other call your application makes. WebSocket subscriptions stream updates as they happen without consuming the per-window REST quota.

**Higher edge limits.** If your application's legitimate usage genuinely needs more than 1,000 REST requests per minute, contact the Reya team. The team reviews on a case-by-case basis depending on your integration profile and traffic patterns.

## When You Hit a Limit

For the **per-wallet limits**, the server rejects the request with a structured error response. The error message identifies which limit was hit (request rate vs. open-order count vs. notional). The connection stays open and other operations continue to work — only the offending request is rejected.

For the **network-edge limit**, the request is rejected at the infrastructure layer with an HTTP `429 Too Many Requests` response. Because this happens before the request reaches the application, there is no structured per-limit detail in the body — just the standard 429 status.

Recommended recovery:

- **Request-rate rejection (per-wallet).** Back off and retry. The simplest pattern is exponential backoff with jitter, starting at a few hundred milliseconds. Because the algorithm is a sliding window, waiting until the oldest counted request ages out (a few seconds, in most cases) is usually enough.
- **Open-order rejection (per-wallet).** Adjust your resting state before retrying. Cancel an older order, wait for fills, or send the new order at a price that crosses (so it fills as IOC instead of resting).
- **Edge `429` rejection.** Wait for the current 60-second window to expire before retrying. If you see these regularly, you're likely polling REST when a WebSocket subscription would serve the same need with no quota cost.

Don't retry tight loops against any rate-limit rejection — you'll keep tripping the same limit and burn server resources for no benefit. A back-off pause of one or two seconds is almost always enough to recover from a transient rejection.

## Best Practices for Integrators

- **Track your own request rate locally.** The server enforces a hard limit, but a well-behaved client paces itself client-side so it never gets close. A token-bucket-style throttle in your client is often the simplest way.
- **Prefer WebSocket Order Entry for high-frequency flow.** The rate limits are identical, but you save the per-request TLS handshake overhead. See the [WebSocket Order Entry API Reference](ws-exec-api-reference.md).
- **Consolidate cancellations.** One `cancelAll` counts as a single mass-cancel request, regardless of how many orders it cancels. Calling `cancelAll` is far more efficient than firing 100 separate `cancelOrder` calls — which would also exhaust your single-cancel budget.
- **Keep an eye on notional, not just order count.** A wallet with 50 small orders is fine on count but may hit the notional cap. Either is a valid cap to hit.
- **Talk to the team if Tier 1 limits don't fit your workload.** Tiers exist precisely because integration profiles vary widely. Market makers, latency-sensitive bots, and high-volume integrations are common reasons for tier upgrades.
