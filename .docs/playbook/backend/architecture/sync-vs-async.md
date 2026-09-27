---
kind: playbook
domain: backend
topics: [synchronous, asynchronous, background-jobs]
authority: guidance
---
# Synchronous vs Asynchronous Work

Choose based on user expectations, execution duration, reliability requirements, and dependency behavior.

## Synchronous request
Use when work is short, predictable, required before the user can continue, and can safely fit within request/runtime limits. Return the final result in the request lifecycle.

Consider timeouts, duplicate submission, transaction boundaries, dependency latency, and what the UI shows while waiting.

## Asynchronous/background job
Use when work may be slow, bursty, retryable, externally dependent, schedulable, batch-oriented, or should continue after the user leaves.

Define:
- durable job/run identity
- queue/persistence model
- state machine
- idempotency
- retry/backoff and terminal failure
- timeout and cancellation
- progress/result storage
- user notification/update transport
- concurrency and rate limits
- observability and dead-letter/manual recovery

## Hybrid
A common pattern is synchronous acknowledgement plus asynchronous execution: create durable state, return an identifier immediately, then stream or query progress. This supports responsive UX without coupling execution lifetime to the browser request.

## Decision questions
- How long can the operation take at p95/p99?
- Must the user wait for completion?
- Can external dependencies fail or throttle?
- Can work be safely retried?
- Must it survive server/browser restart?
- Does the user need cancel, resume, or history?

## Record in project truth
Document lifecycle states, persistence, queue/runtime, retry policy, idempotency boundary, client feedback model, cancellation semantics, recovery, and operational ownership.
