# Reconnection Pattern

Reya's WebSocket APIs — both [Info](websocket-api-reference.md) and [Order Entry](ws-exec-api-reference.md) — share the same reconnection algorithm. Connections can drop for many reasons (network blips, server-side rolling deploys, intermediate proxy timeouts), and a robust client should reconnect automatically rather than treat any disconnect as fatal.

This page covers the algorithm itself (the same on both surfaces) and the post-reconnect steps that differ between the two.

## The Algorithm — Exponential Backoff with Jitter

The connect-loop is identical on both surfaces:

1. **Connect with exponential backoff.** Start the retry delay at `100ms`, double on each failed attempt up to a `30s` cap. This avoids hammering the server during an incident, and avoids being so slow that recovery takes minutes after a transient blip.
2. **Add jitter.** On each retry, perturb the delay by ±50% of its current value. This avoids synchronized reconnect storms when many clients dropped simultaneously (e.g. during a rolling deploy).
3. **Reset on success.** A successful connection returns the delay to the starting `100ms`. The backoff is per-reconnect-attempt, not per-session.

Pseudocode:

```text
backoffMs = 100

loop forever:
  try:
    ws = connect(wsUrl)
    backoffMs = 100                              # reset on success
    onConnected(ws)                              # surface-specific work (see below)
    runUntilDisconnect(ws)
  catch:
    sleep(backoffMs + jitter(-backoffMs / 2, backoffMs / 2))
    backoffMs = min(backoffMs * 2, 30000)
```

The same loop applies whether you're reconnecting to `wss://ws.reya.xyz` (Info) or `wss://ws-exec.reya.xyz` (Order Entry).

For the meaning of WS close codes you'll see on `onclose` (`1000`, `1001`, `1006`, etc.) and what they imply about the disconnect, see [What Happens When the Server Closes the Connection](heartbeats.md#what-happens-when-the-server-closes-the-connection) in the Heartbeats reference.

## After Reconnect

The two surfaces differ on what you should do once the new connection is up. The `onConnected(ws)` step in the pseudocode above is where this surface-specific work happens.

### Info WebSocket

The server holds **no per-connection subscription state** across disconnects. A client that had three channels subscribed before the drop has zero channels subscribed after the new connection opens. After reconnect:

1. **Re-subscribe to every channel the client had active before the disconnect.** Track active subscriptions client-side as part of normal subscribe/unsubscribe handling so the replay is straightforward.
2. **Reconcile missed events.** Channels are best-effort streams — between disconnect and re-subscribe, the client may miss order updates, executions, or balance changes. After reconnect, refresh from REST (e.g. `GET /v2/wallet/{address}/openOrders`, `GET /v2/wallet/{address}/perpExecutions`) before trusting cached state.

Pseudocode for the Info `onConnected` step:

```text
activeSubscriptions = []   # tracked across reconnects

onConnected(ws):
  # Replay subscription state
  for channel in activeSubscriptions:
    ws.send({ type: "subscribe", channel })

  # REST-reconcile any state that may have moved while disconnected
  reconcileFromRest()
```

When subscribing during normal operation, add the channel to `activeSubscriptions` before sending; when unsubscribing, remove it after the `unsubscribed` confirmation arrives.

### Order Entry WebSocket

The Order Entry surface holds **no subscription state and no session-bound identity** — there is nothing to replay on the new connection. The next request authenticates itself via its EIP-712 signature, the same as the first request did.

However, **in-flight requests at the moment of disconnect may have already settled even if you didn't receive the response.** Closing the WebSocket has zero persistent side effects on the order book itself; an in-flight `createOrder` whose response cannot be delivered still settles on-chain (same as a REST timeout). After reconnect:

1. **Verify outcome via REST for any request whose response was not received before disconnect.** Use `GET /v2/wallet/{address}/openOrders` and `GET /v2/wallet/{address}/perpExecutions` to check whether the order is now resting or filled.
2. **Resubmit only if verification confirms the original request did not take effect.** Otherwise you risk a duplicate order with a fresh `clientOrderId`. See [Idempotency (`clientOrderId`)](ws-exec-api-reference.md#idempotency-clientorderid) for how to correlate retried requests safely.

In Order Entry, the `onConnected(ws)` step is therefore typically a no-op for the connection itself — any reconciliation work runs in parallel against REST.
