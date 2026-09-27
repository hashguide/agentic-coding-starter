---
kind: playbook
domain: backend
topics: [idempotency, retries, duplicate-actions]
authority: guidance
---
# Idempotency

Idempotency ensures retries or duplicate delivery do not create unintended duplicate effects.

## Use idempotency when
- creating payments, orders, jobs, messages, or other costly effects
- clients may retry after timeout or network failure
- queues or webhook providers may deliver more than once
- users can double-submit an action
- distributed operations can be replayed

## Approaches

### Natural unique constraint
Use when the domain already has a stable uniqueness key. Enforce it in the database so concurrent requests cannot bypass the rule.

### Idempotency key
Use when the caller can identify one logical operation. Store the key with operation status/result and return the same logical result for safe retries.

### Event/message identity
Use for queues and webhooks. Persist processed event IDs or enforce a domain-level unique effect.

### State-machine guard
Use when transitions themselves define whether an action can be repeated, such as `pending → completed` exactly once.

## Define
- idempotency scope: user, tenant, endpoint, or resource
- key generation/ownership
- retention period
- behavior while the first attempt is still running
- response replay behavior
- conflict behavior when the same key is reused with different input
- transaction/atomicity boundary

## Review
Test simultaneous duplicates, timeout then retry, worker redelivery, process restart, and partial external success. Prefer database/domain guarantees for correctness boundaries rather than in-memory checks.
