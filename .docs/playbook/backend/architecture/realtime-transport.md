---
kind: playbook
domain: backend
topics: [realtime, polling, sse, websocket]
authority: guidance
---
# Realtime Transport

Use this guidance when users need fresh server-side state after the initial request.

## Options

### Request/refresh
Use when updates are uncommon and users can tolerate explicit refresh or normal navigation. It has the smallest operational surface and no persistent connection lifecycle.

### Polling
Use when updates are periodic, a bounded delay is acceptable, and request volume remains reasonable. Define interval, backoff, visibility behavior, terminal-state stopping, and rate limits.

### Server-Sent Events (SSE)
Use when updates primarily flow server → client, such as background-job progress, notifications, monitoring, or incremental results. Define authentication, event IDs, reconnect behavior, replay/current-state recovery, and connection limits.

### WebSocket
Use when both sides send frequent low-latency messages, such as collaboration, presence, multiplayer interaction, or bidirectional realtime control. Define connection lifecycle, authorization, message contracts, ordering, heartbeat/reconnect, scaling, and state recovery.

## Decision questions
- How fresh must the UI be?
- Is communication one-way or bidirectional?
- Can the user leave and return while work continues?
- Is durable state required independently of the connection?
- What happens during reconnect or missed events?
- How many concurrent connections are expected?

## Product consequences
Realtime transport does not replace persistence. Long-running work should normally have durable state that can be queried after reload, reconnect, or navigation. The UI should represent connecting, current, stale/disconnected, reconnecting, terminal, and failed states when applicable.

## Record in project truth
Document the selected transport, event/message schema, authentication model, persistence source of truth, reconnect/recovery behavior, fallback behavior, expected scale, and client state model.

## Review
Verify duplicate/missed event handling, reconnect behavior, authorization, browser lifecycle, cleanup, observability, backpressure/request volume, and whether the chosen freshness actually improves the user journey.
