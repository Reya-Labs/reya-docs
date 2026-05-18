# Heartbeats

Reya's WebSocket APIs (both [Info](websocket-api-reference.md) and [Order Entry](ws-exec-api-reference.md)) use **identical** heartbeat mechanics. This page is the single source of truth for how heartbeats work on both surfaces and what clients need to do — or, more accurately, what they don't need to do.

For most clients, no action is required. Any standards-compliant WebSocket library handles heartbeats automatically at the protocol layer: the server periodically issues protocol-level pings, the client library replies with the corresponding pongs, and the connection stays alive without any application-level involvement. There is no ping handler to write, no pong to send, and no timer to track.

The exception is clients running behind network middleboxes — corporate firewalls or load balancers that drop "idle" connections more aggressively than the server's own idle threshold. Even in those cases, the resolution is a single line of WebSocket-library configuration rather than any application-level work. See [Behind a strict middlebox?](#behind-a-strict-middlebox) for the specifics.

## The Two Layers — Why This Page Exists

There are two completely different "ping/pong" concepts in a WebSocket connection, and confusing them is the most common source of integration bugs. Reya uses one of them for connection health; the other is an optional convenience feature.

### Layer 1 — Protocol-level heartbeat (RFC 6455 control frames)

This is the actual mechanism Reya uses to detect dead connections. It runs **below the application layer**, at the WebSocket protocol itself ([RFC 6455 §5.5.2](https://datatracker.ietf.org/doc/html/rfc6455#section-5.5.2)).

- The server periodically sends a special binary frame called a **PING control frame** (opcode `0x9`).
- The client's WebSocket library — every standards-compliant one — automatically replies with a **PONG control frame** (opcode `0xA`). This happens without any application code running.
- If the client is dead (TCP disconnected, process crashed, network gone), the server gets no PONG. After a configured timeout, the server closes the connection.

The key property: **this layer is invisible to your application code.** You never see a `ping` or `pong` event. Your `onMessage` handler is never invoked. The browser's `WebSocket.onmessage`, Python `websocket-client`'s callback, Node `ws`'s `'message'` event — none of them fire for protocol-level pings or pongs.

### Layer 2 — Application-level JSON ping (optional)

This is a *separate*, *optional* feature. Reya supports a JSON-level `{"type":"ping"}` frame that clients can send if they want explicit round-trip measurement or correlation. The server replies with `{"type":"pong"}`. **This does not contribute to keeping the connection alive** — the protocol-level heartbeat in Layer 1 already handles that. Layer 2 is purely for use cases like:

- Measuring application-level round-trip latency
- Correlating a specific probe to its response (the `id` field round-trips)
- Asserting "the server is alive and processing application messages" (vs just "the TCP socket is up")

If you don't have one of these specific needs, you can ignore Layer 2 entirely.

## How Reya's Servers Are Configured

Both WebSocket APIs run on Bun's WebSocket server with the same two flags:

| Setting | Value | What it does |
|---|---|---|
| `sendPings: true` | enabled | The server periodically sends RFC 6455 PING control frames to every connected client. The client's WS library auto-replies with PONG. |
| `idleTimeout` | 120 seconds | If the server receives **no inbound traffic** (data messages, RFC 6455 PONGs, or anything else) for 120 seconds, it closes the connection. |

These two work together: as long as your client is responsive at the protocol level (which it is, automatically), the auto-pong replies count as inbound traffic and the idle timeout never fires.

If your client truly stops responding — TCP died, process crashed, network dropped — no pong arrives, no other traffic arrives, and after 120 seconds the server closes the socket. Your `onclose` handler fires (with a non-1000 close code), and you reconnect.

## What Your Client Has to Do

**Nothing, for the default case.** The chain of events for a healthy idle connection is:

1. Server sends a protocol-level PING.
2. Your WebSocket library's networking layer receives the PING, generates a PONG, sends it back. None of this surfaces to your application code.
3. The PONG arrives at the server. The server's idle counter resets.
4. Repeat every ~half of `idleTimeout`.

Your application doesn't see steps 1–4. It only sees data messages you care about. Your `onmessage` / `on_message` / etc. handler is never invoked for any of this.

## Behind a Strict Middlebox?

Some networks (corporate firewalls, mobile carriers, aggressive load balancers) drop "idle" TCP connections after ~30–60 seconds even if both ends would prefer to keep them alive. Reya's server-sent pings happen well within the server's 120-second timeout, but they're paced for the server's needs — not for the most aggressive middleboxes on the planet.

If you encounter spurious disconnects through such a network, configure your **client** to send its own protocol-level pings at a shorter interval. This is one line of config in every common WebSocket library:

**Python (`websocket-client`):**
```python
ws.run_forever(ping_interval=20, ping_timeout=10)
```

**Python (`websockets`, async):**
```python
async with websockets.connect(url, ping_interval=20, ping_timeout=10) as ws:
    ...
```

**Node (`ws`):**
```javascript
setInterval(() => ws.ping(), 20_000);  // ws library auto-handles pong replies
```

**Rust (`tokio-tungstenite`):**
```rust
// Send a Ping control frame manually
ws.send(Message::Ping(vec![])).await?;
```

**Browser (vanilla `WebSocket`):**
The browser handles protocol-level pings internally and exposes no API to send them from JavaScript. If your browser-based client gets dropped by a middlebox, the only knob is application-level: send a `{"type":"ping"}` JSON frame periodically. See the next section.

The "20 seconds" figure above is conservative — fast enough for almost any middlebox, light enough to be invisible in traffic volume. Adjust if your environment requires something more aggressive.

## The Optional Application-Level Probe

If you need it, here's how Layer 2 looks on the wire:

### Client sends a ping

```json
{
  "type": "ping",
  "id": "probe-001"
}
```

- `type`: required, must be the literal string `"ping"`.
- `id`: optional. A free-form string you choose. Use it if you want to correlate this specific probe with its reply; omit it if you just want a round-trip sanity check.

### Server replies with a pong

```json
{
  "type": "pong",
  "id": "probe-001"
}
```

- `type`: literal string `"pong"`.
- `id`: echoed back **if** the client sent one. If the client's ping had no `id`, the pong has no `id`.

That's the entire payload — there are no other fields, no `timestamp`, no `serverTimeMs`, nothing else. The server doesn't include its own time and doesn't echo any client time.

### When to use Layer 2 vs trust Layer 1

| Need | Solution |
|---|---|
| "Keep my connection alive" | Layer 1 (automatic, do nothing) |
| "Detect a half-dead server / unresponsive backend" | Layer 2 (send a JSON ping, expect a JSON pong) |
| "Measure round-trip latency from my application code" | Layer 2 (timestamp your ping send, subtract from pong receive) |
| "Correlate a probe with its specific reply" | Layer 2 (use the `id` field) |
| "Survive an aggressive middlebox" | Configure your WS library's protocol-level `ping_interval` (Layer 1, client-initiated) |

Note that Layer 2 is purely additive — using it doesn't disable or replace the Layer 1 mechanism. Both run in parallel.

## What Happens When the Server Closes the Connection

The server can close a connection for several reasons; you'll see different close codes depending on cause:

| Close code | Meaning | Recommended client action |
|---|---|---|
| `1000` | Normal closure (you or the server called close cleanly) | Reconnect if you still want a connection |
| `1001` | Server going away (e.g. during a rolling deploy / pod restart) | Reconnect after a short backoff |
| `1006` / `1011` / others | Idle timeout, server error, abrupt closure | Reconnect with exponential backoff |

In all cases, the right pattern is the same: reconnect with backoff, replay your subscription state (for Info) or your in-flight order state (for Order Entry — but note in-flight orders are independently durable on-chain or in the matching engine, so you don't typically need to replay anything).

For the specific reconnection algorithm, see [Reconnection Pattern](reconnection-pattern.md) — the same algorithm applies to both the Info and Order Entry WebSocket APIs.

### Server-Side Graceful Shutdown

During a server-side rolling deploy or pod restart, the server closes connections with WS close code `1001 SERVER_SHUTTING_DOWN` after a drain period (currently 10 seconds) in which:

1. The `/ready` health endpoint returns `503` so the load balancer stops sending new connections to this pod.
2. In-flight requests are allowed to complete.
3. Idle connections are closed first; busy connections wait for their in-flight handler to finish.

After the drain timeout, any still-open connections are force-closed with `1001`. Clients should treat `1001` as a soft reconnect signal — the next pod is already accepting new connections.

This applies to both the Info and Order Entry WebSocket APIs. The drain period matters more in practice for Order Entry (where an in-flight `createOrder` whose response is lost can still settle on-chain — see [Reconnection Pattern](reconnection-pattern.md#order-entry-websocket) for how to reconcile), but the close-code sequence is the same on both surfaces.

## Frequently Asked Questions

**Q: Do I need to implement a ping handler in my client?**
No. The protocol-level heartbeat is handled by your WebSocket library automatically. You only need an application-level ping handler if you choose to use the Layer 2 probe.

**Q: How do I know if the server-side pings are actually firing?**
You generally don't, and you don't need to. They happen at a layer below your application code. If your connection survives 5+ minutes of application-level silence, the pings are firing correctly. If it dies after exactly 120 seconds of silence and you didn't see any application-level activity, your client's WS library isn't responding to protocol pings — which is unusual but worth investigating.

**Q: Can I send `{"type":"pong"}` from my client?**
No, and you shouldn't. The server has no handler for client-initiated JSON pong frames — sending one will result in an "Invalid type" / `UNKNOWN_TYPE` error. The only `pong` direction is server→client, in response to a client-initiated `{"type":"ping"}` (Layer 2).

**Q: Does the optional `id` on a ping need to be unique?**
No. It's just a string you choose. If you send multiple pings in flight at once and want to match each pong to its ping, use unique `id`s. If you only ever have one ping in flight at a time, you can reuse the same `id` or omit it entirely.

**Q: What's the `timestamp` field I see in the spec?**
The AsyncAPI spec declares `timestamp?: integer` on both `PingMessagePayload` and `PongMessagePayload`. Today the server doesn't read or set it — it's reserved for potential future use (e.g. server-side time for clock-sync). You can safely ignore it.
