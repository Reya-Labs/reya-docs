# WebSocket Order Entry API Reference

## Overview

The Reya DEX WebSocket Order Entry API v2 is a request/response surface for placing and cancelling orders over a persistent WebSocket connection. It carries the same operations and payload bodies as the REST `/v2` endpoints (`POST /v2/createOrder`, `POST /v2/cancelOrder`, `POST /v2/cancelAll`), with lower per-operation overhead and id-correlated responses on the same channel.

Payload bodies are reused verbatim from REST. Request and response envelopes are id-correlated; the server replies on the same connection with a frame carrying the same `id` the client sent.

This surface is **order-entry only**. For real-time market data, position updates, and fill streaming, see the [WebSocket Market Data API Reference](websocket-api-reference.md). The recommended Market Maker integration runs both connections in parallel: this surface for order entry, the streaming surface for read-side fanout.

## Server Endpoints

### Production Environment

* **URL**: `wss://ws-exec.reya.xyz`
* **Protocol**: WSS
* **Description**: Production WebSocket order entry server

### Staging Environment

* **URL**: `wss://ws-exec-staging.reya.xyz`
* **Protocol**: WSS
* **Description**: Staging WebSocket order entry server for pre-production testing

### Test Environment

* **URL**: `wss://ws-exec-testnet.reya.xyz`
* **Protocol**: WSS
* **Description**: Test WebSocket order entry server (cronos)

## Connection & Auth Model

The connection itself is **anonymous** — no handshake, no login, no API key. There is no concept of a session-bound wallet identity.

Authentication is **per-frame**: every order-bearing request carries an EIP-712 signature in its `payload` (`signature`, `nonce`, `signerWallet`, `expiresAfter`). The server validates the signature against the order contents on every request — identical to the REST `/v2/createOrder` etc. body shape. See [Signatures and Nonces](rest-api-reference/signatures-and-nonces.md) for the signing model; both transports use the same scheme and the same Python SDK helpers.

A consequence of per-frame authentication is that a single WebSocket connection can carry orders signed by **multiple different `signerWallet` values** — useful for market makers operating multiple subaccounts on one connection.

## Message Envelope

All WebSocket messages follow a standardized envelope structure with a `type` discriminator and a client-chosen `id` for correlation.

### Request Envelope (Client → Server)

```json
{
  "type": "createOrder",
  "id": "req-7f3c1a",
  "payload": { /* operation-specific request body */ }
}
```

#### Components

* **type** (string, required): One of `createOrder`, `cancelOrder`, `cancelAll`, `ping`, `pong`.
* **id** (string, required): Client-chosen correlation identifier. Must be unique across in-flight requests on the connection — see [In-Flight `id` Uniqueness](#in-flight-id-uniqueness) below.
* **payload** (object, required for `createOrder` / `cancelOrder` / `cancelAll`): Operation-specific request body, byte-identical to the corresponding REST endpoint's request body.

### Response Envelope (Server → Client)

```json
{
  "type": "createOrder",
  "id": "req-7f3c1a",
  "ok": true,
  "payload": { /* operation-specific success body */ }
}
```

or, on failure:

```json
{
  "type": "createOrder",
  "id": "req-7f3c1a",
  "ok": false,
  "error": {
    "error": "INPUT_VALIDATION_ERROR",
    "message": "qty is missing"
  }
}
```

#### Components

* **type** (string, required): Echoes the request `type`.
* **id** (string, required): Echoes the request `id`.
* **ok** (boolean, required): `true` for success, `false` for failure.
* **payload** (object, required when `ok = true`, forbidden when `ok = false`): Operation-specific success body, byte-identical to the REST `200` response body.
* **error** (object, required when `ok = false`, forbidden when `ok = true`): See [Error Catalog](#error-catalog) for the shape and possible codes.

### Heartbeat Envelopes

#### Server Ping (Server → Client, every 5s)

```json
{
  "type": "ping"
}
```

#### Client Pong (Client → Server, reply to server ping)

```json
{
  "type": "pong"
}
```

#### Client Ping (Client → Server, optional liveness probe)

```json
{
  "type": "ping",
  "id": "probe-001"
}
```

#### Server Pong (Server → Client, reply to client ping)

```json
{
  "type": "pong",
  "id": "probe-001",
  "server_time_ms": 1747927089946
}
```

The server-emitted pong always carries `server_time_ms` (server wall-clock at pong emission, useful for clock-sync and RTT measurement); the client-emitted pong never does. See [Heartbeat Semantics](#heartbeat-semantics) for timeout behavior.

### Top-Level Error Envelope (Server → Client)

The server emits a top-level `error` envelope when it cannot parse a request at all (malformed JSON, unknown `type`, in-flight `id` collision, internal failure). Connection stays open. Operation-specific errors instead come back as `{ ok: false, error }` on the corresponding response envelope correlated by `id`.

```json
{
  "type": "error",
  "id": "req-7f3c1a",
  "error": {
    "error": "DUPLICATE_REQUEST_ID",
    "message": "Request id already in-flight on this connection"
  }
}
```

* **id** is present when the offending request carried one (e.g. for `DUPLICATE_REQUEST_ID`); absent for frame-level errors (e.g. unparseable JSON) where the server has no id to echo.

## Operations Reference

### `createOrder`

**Purpose**: Place a spot LIMIT GTC order, a perp IOC order, or a perp GTC/SL/TP conditional order. Identical body and semantics to REST `POST /v2/createOrder`.

**Request Envelope** (spot LIMIT GTC):

```json
{
  "type": "createOrder",
  "id": "req-7f3c1a",
  "payload": {
    "exchangeId": 2,
    "symbol": "WETHRUSD",
    "accountId": 10000000002,
    "isBuy": true,
    "limitPx": "1",
    "qty": "0.001",
    "orderType": "LIMIT",
    "timeInForce": "GTC",
    "signature": "0x...",
    "nonce": "1778601294274124",
    "signerWallet": "0x869d6494fe32B96F93F78F9c4B7aAf30eeC01C1F",
    "expiresAfter": 1747927089,
    "clientOrderId": 1778601294274124
  }
}
```

**Request Envelope** (perp IOC):

```json
{
  "type": "createOrder",
  "id": "req-7f3c1b",
  "payload": {
    "exchangeId": 2,
    "symbol": "ETHRUSDPERP",
    "accountId": 8017,
    "isBuy": true,
    "limitPx": "40000",
    "qty": "0.01",
    "orderType": "LIMIT",
    "timeInForce": "IOC",
    "reduceOnly": false,
    "signature": "0x...",
    "nonce": "...",
    "signerWallet": "0x869d6494fe32B96F93F78F9c4B7aAf30eeC01C1F",
    "expiresAfter": 1747927089
  }
}
```

**Success Response**:

```json
{
  "type": "createOrder",
  "id": "req-7f3c1a",
  "ok": true,
  "payload": {
    "status": "OPEN",
    "orderId": "1864998629727535104",
    "clientOrderId": 1778601294274124
  }
}
```

**Error Response**:

```json
{
  "type": "createOrder",
  "id": "req-7f3c1a",
  "ok": false,
  "error": {
    "error": "INPUT_VALIDATION_ERROR",
    "message": "limitPx is required"
  }
}
```

<details>

<summary><strong>Data Type — CreateOrderRequest payload</strong></summary>

* `exchangeId` (integer, required): Reya exchange identifier. Currently always `2`.
* `symbol` (string): Trading symbol (e.g. `WETHRUSD`, `ETHRUSDPERP`).
* `accountId` (integer, required): Reya account ID placing the order.
* `isBuy` (boolean, required): `true` for a buy, `false` for a sell.
* `limitPx` (string, required): Limit price as a decimal string.
* `qty` (string): Order quantity as a decimal string.
* `orderType` (string, required): `LIMIT`, `TP` (take-profit), or `SL` (stop-loss).
* `timeInForce` (string): `IOC` or `GTC`. Required for `LIMIT` orders.
* `triggerPx` (string): Trigger price. Required for `TP` / `SL` orders.
* `reduceOnly` (boolean): Whether the order is reduce-only. Required for perp IOC orders.
* `signature` (string, required): EIP-712 signature over the order. See [Signatures and Nonces](rest-api-reference/signatures-and-nonces.md).
* `nonce` (string, required): Order nonce.
* `signerWallet` (string, required): Address that produced the signature.
* `expiresAfter` (integer): Expiration timestamp in seconds since epoch. Required for perp IOC orders and all spot orders.
* `clientOrderId` (integer, optional): Echoed back in the response; useful for client-side correlation independent of the server-issued `orderId`.

</details>

<details>

<summary><strong>Data Type — CreateOrderResponse payload</strong></summary>

* `status` (string, required): One of `OPEN`, `FILLED`, `CANCELLED`, `REJECTED`.
* `execQty` (string, optional): Executed quantity in this order update.
* `cumQty` (string, optional): Total executed quantity across all fills where the order is active.
* `orderId` (string, optional): Server-issued order ID. Present for all order types except perp IOC (which fills/voids on-chain in the same call and has no resting state).
* `clientOrderId` (integer, optional): Echoes the request's `clientOrderId`.

</details>

### `cancelOrder`

**Purpose**: Cancel a previously placed order by `orderId` (or `clientOrderId` for spot). Identical body and semantics to REST `POST /v2/cancelOrder`.

**Request Envelope**:

```json
{
  "type": "cancelOrder",
  "id": "req-7f3c1c",
  "payload": {
    "orderId": "1864998629727535104",
    "accountId": 10000000002,
    "symbol": "WETHRUSD",
    "signature": "0x...",
    "nonce": "1778601294356211",
    "expiresAfter": 1747927089
  }
}
```

**Success Response**:

```json
{
  "type": "cancelOrder",
  "id": "req-7f3c1c",
  "ok": true,
  "payload": {
    "status": "CANCELLED",
    "orderId": "1864998629727535104"
  }
}
```

<details>

<summary><strong>Data Type — CancelOrderRequest payload</strong></summary>

* `orderId` (string): Internal matching engine order ID to cancel. Provide either `orderId` or `clientOrderId`.
* `clientOrderId` (integer): Client-provided order ID to cancel. Provide either `orderId` or `clientOrderId`.
* `accountId` (integer): Account ID that owns the order. Required for spot markets.
* `symbol` (string): Market symbol for the order. Required for spot market orders.
* `signature` (string, required): EIP-712 signature over the cancellation.
* `nonce` (string): Cancel nonce. Required for spot.
* `expiresAfter` (integer): Expiration timestamp. Required for spot.

</details>

<details>

<summary><strong>Data Type — CancelOrderResponse payload</strong></summary>

* `status` (string, required): Always `CANCELLED` on success.
* `orderId` (string, required): The cancelled order ID.
* `clientOrderId` (integer, optional): Echoes the request's `clientOrderId`.

</details>

### `cancelAll`

**Purpose**: Mass-cancel all open orders for an account on a given (spot) market, or across all markets if `symbol` is omitted. Identical body and semantics to REST `POST /v2/cancelAll`. Perp mass-cancel is not supported and returns the same not-supported error REST returns.

**Request Envelope**:

```json
{
  "type": "cancelAll",
  "id": "req-7f3c1d",
  "payload": {
    "accountId": 10000000002,
    "symbol": "WETHRUSD",
    "signature": "0x...",
    "nonce": "1778601294501222",
    "expiresAfter": 1747927089
  }
}
```

**Success Response**:

```json
{
  "type": "cancelAll",
  "id": "req-7f3c1d",
  "ok": true,
  "payload": {
    "cancelledCount": 3
  }
}
```

<details>

<summary><strong>Data Type — MassCancelRequest payload</strong></summary>

* `accountId` (integer, required): Account ID to cancel orders for.
* `symbol` (string, optional): Symbol to cancel orders for. If omitted, cancels all orders for the account across all markets.
* `signature` (string, required): EIP-712 signature.
* `nonce` (string, required): Mass-cancel nonce.
* `expiresAfter` (integer, required): Expiration timestamp.

</details>

<details>

<summary><strong>Data Type — MassCancelResponse payload</strong></summary>

* `cancelledCount` (integer, required): Number of orders that were cancelled.

</details>

### `ping` / `pong`

**Purpose**: Heartbeat. Both client and server may initiate a ping; the receiver replies with a pong on the same connection. See [Heartbeat Semantics](#heartbeat-semantics) for timeout behavior and [Heartbeat Envelopes](#heartbeat-envelopes) for the wire shape.

A client-initiated ping is **optional** — used for clock-sync or active RTT measurement. Server-initiated pings are mandatory: the client must reply within 5 seconds or the connection is closed with code `4002 HEARTBEAT_TIMEOUT`.

## Error Catalog

Every error envelope (both per-operation `{ok: false, error}` and top-level `error`) carries a `RequestError`-shaped object:

```json
{
  "error": "INPUT_VALIDATION_ERROR",
  "message": "qty is missing"
}
```

The `error` field is one of the codes below. Per-operation responses use codes from the **Trade Handler** group (shared 1:1 with REST). Top-level `error` envelopes additionally use codes from the **Framing Layer** group; these only make sense for a streamed envelope protocol and never appear in REST responses.

### Trade Handler Codes (shared with REST)

| Code | When emitted | Client action |
|---|---|---|
| `SYMBOL_NOT_FOUND` | The `symbol` in the payload doesn't resolve to a known market. | Refresh market definitions via REST `GET /v2/marketDefinitions`. |
| `NO_ACCOUNTS_FOUND` | The `accountId` doesn't exist or isn't owned by the signer. | Verify account configuration. |
| `NO_PRICES_FOUND_FOR_SYMBOL` | Stork oracle has no recent price for the symbol — usually transient at startup. | Retry after a short backoff. |
| `INPUT_VALIDATION_ERROR` | Generic body-validation failure (missing field, wrong type, illegal value). | Fix the request body; consult the human-readable `message`. |
| `CREATE_ORDER_OTHER_ERROR` | Generic createOrder failure not covered by a more specific code (includes on-chain reverts surfaced through the relayer). | Read `message` for the underlying reason; for perp IOC, also check that the limit price can be filled within the on-chain price-limit check. |
| `CANCEL_ORDER_OTHER_ERROR` | Generic cancelOrder failure not covered by a more specific code. | Read `message`. |
| `ORDER_DEADLINE_PASSED_ERROR` | `expiresAfter` is in the past. | Re-sign with a fresh deadline. |
| `ORDER_DEADLINE_TOO_HIGH_ERROR` | `expiresAfter` is too far in the future (anti-abuse cap). | Use a shorter deadline (typically <= 24h for spot GTC, <= 60s for IOC). |
| `INVALID_NONCE_ERROR` | Nonce is not strictly monotonic for this signer, or was already used. | Re-sign with a fresh monotonic nonce. |
| `UNAVAILABLE_MATCHING_ENGINE_ERROR` | The matching engine is unavailable (transient). | Retry after a short backoff. |
| `UNAUTHORIZED_SIGNATURE_ERROR` | The recovered signer is not authorized to act on the `accountId`. | Verify the signer wallet is in the account's permissioned-addresses list on-chain. |
| `NUMERIC_OVERFLOW_ERROR` | A numeric field exceeds the allowed uint64 / int256 range. | Fix the request body. |

### Framing Layer Codes (top-level `error` envelope only)

These codes appear **only** in the top-level `error` envelope, never inside a per-operation `{ok: false}` response.

| Code | When emitted | Client action |
|---|---|---|
| `MALFORMED_JSON` | The frame body could not be parsed as JSON, or required envelope fields (`type`, `id`) are missing. Connection stays open. | Fix the client serializer. |
| `UNKNOWN_TYPE` | The frame's `type` field is not one of the accepted values. Connection stays open. | Verify the request `type`. |
| `DUPLICATE_REQUEST_ID` | The frame's `id` is already in-flight on this connection — i.e., the client sent a new request with an `id` whose response hasn't yet been emitted. Connection stays open; the new request is rejected, the original is unaffected. | Use a fresh `id` for each request (UUIDs work). |
| `INTERNAL` | The server hit an internal failure handling the frame. Connection stays open. | Retry; if the problem persists, contact support with the `id`. |

## Data Types & Schemas

### Enumeration Types

<details>

<summary><strong>OrderType</strong></summary>

* `LIMIT` — Limit order (with `timeInForce` = `IOC` or `GTC`).
* `TP` — Take-profit conditional order (perp).
* `SL` — Stop-loss conditional order (perp).

</details>

<details>

<summary><strong>TimeInForce</strong></summary>

* `IOC` — Immediate or Cancel. Required for perp IOC orders.
* `GTC` — Good Till Cancelled. Used for spot LIMIT orders.

</details>

<details>

<summary><strong>OrderStatus</strong></summary>

* `OPEN` — Order is resting in the book.
* `FILLED` — Order is fully filled.
* `CANCELLED` — Order is cancelled.
* `REJECTED` — Order was rejected by the matching engine.

</details>

For the complete enumeration of `RequestErrorCode` (per-operation errors) and `WsExecRequestErrorCode` (top-level errors), see the [Error Catalog](#error-catalog) above.

## Connection Management

### Heartbeat Semantics

The server sends a `ping` frame every 5 seconds. The client must reply with a `pong` frame within 5 seconds of receiving the ping. If the server observes no pong within `HEARTBEAT_INTERVAL_MS + HEARTBEAT_TIMEOUT_MS` (10 seconds total) since the last successful pong, it closes the connection with WS close code `4002 HEARTBEAT_TIMEOUT`. Clients should treat `4002` as "reconnect", not "fatal".

A client may also send a `ping` of its own at any time. The server replies with a pong containing `server_time_ms` (server wall-clock at emission, milliseconds since epoch). Useful for clock-sync and active RTT measurement.

### Graceful Shutdown

During a server-side rolling deploy or pod restart, the server closes connections with WS close code `1001 SERVER_SHUTTING_DOWN` after a drain period (currently 10 seconds) in which:

1. The `/ready` health endpoint returns `503` so the load balancer stops sending new connections to this pod.
2. In-flight requests are allowed to complete.
3. Idle connections are closed first; busy connections wait for their in-flight handler to finish.

After the drain timeout, any still-open connections are force-closed with `1001`. Clients should treat `1001` as a soft reconnect signal.

### Reconnection Pattern

Closing the WebSocket has **zero persistent side effects** on the order book — an in-flight `createOrder` whose response cannot be delivered still settles on-chain (same as a REST timeout). On reconnect, clients should:

1. Re-establish the WebSocket connection with exponential backoff (start at 100ms, double up to a 30s cap, with jitter).
2. Resume normal operation. There is no subscription state to replay (unlike the market-data WebSocket) and no session to restore.
3. For any request whose response was not received before disconnect, verify outcome via REST `GET /v2/wallet/{address}/openOrders` and `GET /v2/wallet/{address}/perpExecutions` before resubmitting — otherwise you risk a duplicate order with a fresh `clientOrderId`.

Pseudocode:

```text
backoffMs = 100
loop:
  try:
    connect()
    backoffMs = 100         # reset on success
    runUntilDisconnect()
  catch (anyError):
    sleep(backoffMs + jitter(0..backoffMs/2))
    backoffMs = min(backoffMs * 2, 30000)
```

### In-Flight `id` Uniqueness

Each `id` must be unique across **in-flight** requests on the connection. Once the server has emitted the corresponding response envelope, the `id` is free to reuse. Sending a fresh request with the same `id` as an in-flight one triggers a top-level `DUPLICATE_REQUEST_ID` error envelope; the original in-flight request is unaffected.

In practice, clients should generate a fresh `id` for every request (e.g. UUIDv4 or a monotonic counter prefixed with a session token).

### Idempotency (`clientOrderId`)

For `createOrder`, the optional `clientOrderId` field is echoed back unchanged in the response and in any subsequent order-update events on the market-data WebSocket. Use it for client-side correlation independent of the server-issued `orderId` — particularly useful when the response envelope is lost mid-flight and the client must reconcile state from market-data updates after reconnect.

`clientOrderId` does not provide server-side deduplication: two requests with the same `clientOrderId` will be treated as two separate orders.

## Signatures and Nonces

All `createOrder`, `cancelOrder`, and `cancelAll` payloads carry an EIP-712 `signature` over the order contents. The shape of the signed message is identical to the REST `/v2` endpoints — see [Signatures and Nonces](rest-api-reference/signatures-and-nonces.md) for the canonical reference. The Python SDK provides the helpers `sign_raw_order`, `sign_cancel_order_spot`, and `sign_mass_cancel` to produce these signatures correctly.

The WebSocket transport does **not** add or change any signing requirement. Frames are signed by your wallet (or a permissioned trading key) at the payload level; the WebSocket connection itself is unauthenticated.

## Differences vs REST

The WebSocket Order Entry surface is functionally equivalent to the corresponding REST endpoints, with transport-level differences:

| Concern | REST | WebSocket Order Entry |
|---|---|---|
| Request body | `CreateOrderRequest` / `CancelOrderRequest` / `MassCancelRequest` | Same — embedded as `payload` |
| Response body | `CreateOrderResponse` / `CancelOrderResponse` / `MassCancelResponse` | Same — embedded as `payload` on `ok: true` |
| Success / failure | HTTP `200` vs `400` / `500` | `ok: true` vs `ok: false` inside the frame |
| Error body | `RequestError` | Same — embedded as `error` on `ok: false` |
| Correlation | HTTP request/response pairing | Client-supplied `id` field |
| Heartbeat | n/a (stateless) | Bidirectional ping/pong, 5s interval, 10s timeout |
| Auth | EIP-712 signature in body | Same EIP-712 signature in same body |
| Idempotency | `clientOrderId` echoed | Same |
| Connection | Per-request | Persistent, multiplexed |

**When to use which:**

* **REST** — One-off requests, low frequency, simpler client integration, no need to maintain a long-lived connection.
* **WebSocket Order Entry** — High-frequency order submission, lower per-request overhead (no TLS handshake per request), latency-sensitive integrations. Recommended for Market Makers running together with the [WebSocket Market Data API](websocket-api-reference.md).

## Python SDK Example

A worked end-to-end example is included in the [Reya Python SDK](https://github.com/Reya-Labs/reya-python-sdk) at [`examples/ws_exec/mvp.py`](https://github.com/Reya-Labs/reya-python-sdk/blob/main/examples/ws_exec/mvp.py). It demonstrates:

1. Spot LIMIT GTC `createOrder` (resting in the book)
2. Spot `cancelOrder` (cancels the order from step 1)
3. Spot `cancelAll` (opens N orders then mass-cancels them)
4. Perp IOC `createOrder` (fills at the current mark with a loose limit)

Run with:

```bash
poetry shell
python -m examples.ws_exec.mvp
```

See the script's accompanying README for prerequisites (`.env` setup, funded test accounts on cronos).
