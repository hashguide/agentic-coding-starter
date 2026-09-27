---
kind: playbook
domain: product
topics: [product, engineering, principles]
authority: guidance
---

# Engineering Principles

Five principles govern every technical decision in this playbook. They are not abstract values — each is a specific decision filter. When two approaches seem equivalent, run them through these principles. When a deadline creates pressure to shortcut, these principles name the cost explicitly so the trade is made consciously, not accidentally.

## KISS — Keep It Simple, Stupid

**Definition:** The simplest solution that correctly handles the known cases is the right solution. Complexity introduced ahead of need is a liability, not an asset.

**Code example:**

```typescript
// Over-engineered for a two-case problem
class NotificationDispatcherFactory {
  static create(type: string): NotificationDispatcher {
    return registry.get(type) ?? new DefaultDispatcher();
  }
}

// Simple, correct, extensible when needed
function sendNotification(userId: string, message: string) {
  return db.notifications.create({ userId, message, read: false });
}
```

**When this conflicts with another principle:** KISS conflicts with DRY when removing duplication requires a non-obvious abstraction. Prefer the duplication until the pattern is clear.

## DRY — Don't Repeat Yourself

**Definition:** Every piece of knowledge has a single, authoritative representation. Duplication creates consistency risk — a fact stated in two places will eventually disagree.

**Code example:**

```typescript
// Bad: validation logic duplicated
function createUser(email: string) {
  if (!email.includes('@')) throw new Error('Invalid email');
  // ...
}

function updateEmail(email: string) {
  if (!email.includes('@')) throw new Error('Invalid email');
  // ...
}

// Good: single source of truth
const emailSchema = z.string().email();

function createUser(email: string) {
  emailSchema.parse(email);
  // ...
}

function updateEmail(email: string) {
  emailSchema.parse(email);
  // ...
}
```

**When DRY abstraction is premature:** Abstracting after one occurrence (or two similar-looking occurrences with different underlying semantics) usually creates the wrong abstraction. Apply the Rule of Three: wait for a third genuine repetition before abstracting. See [refactoring triggers](../quality/code/refactoring-triggers.md).

## YAGNI — You Aren't Gonna Need It

**Definition:** Do not implement something because you might need it in the future. Implement it when the need is real and present.

**Code example:**

```typescript
// YAGNI violation — "supporting" multiple storage backends before any exist
interface StorageBackend {
  upload(file: Buffer): Promise<string>;
  delete(key: string): Promise<void>;
}

class S3Backend implements StorageBackend { /* ... */ }
class GCSBackend implements StorageBackend { /* ... */ } // added "just in case"

// Correct — implement what exists now
async function uploadFile(file: Buffer): Promise<string> {
  const key = `uploads/${randomUUID()}`;
  await s3.putObject({ Bucket: process.env.S3_BUCKET!, Key: key, Body: file });
  return key;
}
```

**When YAGNI gets violated under deadline pressure:** Features get merged with "temporary" workarounds that become permanent. Name the debt at the time of introduction — open a ticket, add a `// TODO(issue-#):` comment — so it's tracked rather than forgotten.

## SSOT — Single Source of Truth

**Definition:** Any given fact — data, configuration, schema, UI state — has exactly one canonical location. Everything else derives from it.

**Applies to data:** User subscription status lives in the database. It is not cached in the session without an invalidation strategy. It is not duplicated in a separate "user_status" table without a sync mechanism.

**Applies to UI state:** Server data is not copied into global client state (see [state management](../frontend/state-management.md)). The server is the source of truth; the client caches queries, not facts.

**Applies to configuration:** Environment variable schemas are defined once and validated at startup (see [env config](../backend/runtime/env-config.md)).

**When this conflicts with another principle:** Caching is a deliberate, documented violation of SSOT — you accept a window of staleness for a performance gain. Make that tradeoff explicit and bounded.

## Fail Fast

**Definition:** When invalid state is detected, abort immediately with a clear error rather than continuing with corrupted assumptions. Failing early produces a clean, proximate error. Failing late produces a confusing, distal one.

**Code example:**

```typescript
// Fail late — hard to debug
function processOrder(orderId: string) {
  const order = getOrder(orderId); // returns undefined if not found
  const total = order.items.reduce(/* ... */); // TypeError: Cannot read properties of undefined
}

// Fail fast — clear error at the boundary
function processOrder(orderId: string) {
  const order = getOrder(orderId);
  if (!order) throw new NotFoundError(`Order ${orderId} not found`);
  const total = order.items.reduce(/* ... */);
}
```

**At startup:** The application validates all required configuration at boot time and refuses to start with a clear, human-readable error if any required environment variable is missing or malformed. See [env config](../backend/runtime/env-config.md).

**When Fail Fast conflicts with graceful degradation:** For external dependencies (third-party APIs, non-critical services), controlled degradation is preferable to crashing. For internal invariants and required configuration, fail fast without exception.

## Related Documents

- [Code quality overview](../quality/code/overview.md)
- [Refactoring triggers](../quality/code/refactoring-triggers.md)
- [Error handling](../backend/runtime/error-handling.md)
- [Environment configuration](../backend/runtime/env-config.md)
