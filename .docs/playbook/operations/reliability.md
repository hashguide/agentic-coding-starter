---
kind: playbook
domain: operations
topics: [reliability, resilience, recovery]
authority: guidance
---
# Reliability

Reliability requirements should reflect the user impact of failure and the product's stage.

## Define service expectations
For important capabilities establish expected availability, acceptable latency, data durability, recovery expectations, and dependency assumptions. Use explicit SLOs or simpler targets when they improve engineering decisions.

## Design for failure
For each critical dependency or workflow define:
- timeout behavior
- retry/backoff policy
- idempotency requirements
- circuit/degraded behavior where useful
- queue/backlog handling
- persistence and restart recovery
- user-visible failure/recovery state
- operator visibility and manual recovery path

## Data recovery
Define backup coverage, restore procedure, recovery point expectation, recovery time expectation, and restore testing for data that cannot simply be recreated.

## Graceful degradation
Identify which capabilities can remain usable when email, analytics, AI, payments, search, realtime transport, or another dependency is unavailable. Preserve core user work where feasible.

## Review
Exercise dependency timeout, rate limit, partial failure, queue backlog, process restart, database outage/reconnect, duplicate delivery, and recovery. Verify alerts correspond to user-impacting conditions rather than raw noise.

Record reliability requirements in project NFRs and delivery/operations docs, and feature-specific recovery behavior in feature specifications.
