---
kind: playbook
domain: experience
topics: [async, background-job, progress, polling, sse, websocket, experience, interaction, long, running, actions]
authority: guidance
---
# Long-Running Actions

Model long-running work as a persistent operation rather than a blocked request when users may wait, navigate away, reconnect, or need history.

## Inline request
Use when work reliably completes within normal interaction latency and the result is needed immediately.

## Background job
Use when work is slow, retryable, externally dependent, resource-intensive, schedulable, or should survive the initiating request. Return an operation identifier promptly and persist its state.

## Update transport
- **Polling:** suitable when updates are infrequent and a small delay is acceptable.
- **Server-Sent Events:** suitable for primarily server-to-client progress and status streams.
- **WebSocket:** suitable for frequent bidirectional realtime interaction.
- **Push/email/in-app notification:** suitable when completion may occur after the user leaves the active session.

Define queued, running, waiting, succeeded, failed, cancelled and reconnect behavior as applicable. Ensure the user can understand what is happening without remaining on the initiating screen.
