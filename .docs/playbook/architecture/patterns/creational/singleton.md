---
kind: playbook
domain: architecture
topics: [architecture, patterns, creational, singleton]
authority: guidance
---

# Singleton Pattern

The Singleton pattern ensures a class has only one instance throughout the application's lifetime and provides a global access point to that instance. It is one of the most commonly used patterns and one of the most commonly misused ones. The legitimate use cases are narrow; the misuse cases are broad.

## Definition

A singleton is a class that constructs itself only once, stores the instance internally, and returns the same instance on every subsequent call to `getInstance()` (or equivalent).

## Justified Use Case

A database connection pool must be shared — creating a new pool per request exhausts connections:

```typescript
import { Pool } from 'pg';

let pool: Pool | null = null;

export function getDbPool(): Pool {
  if (!pool) {
    pool = new Pool({
      connectionString: process.env.DATABASE_URL,
      max: 10,
      idleTimeoutMillis: 30_000,
    });
  }
  return pool;
}
```

In Node.js module caching achieves the singleton effect naturally — a module is evaluated once and the exported value is the same object on every `import`. This is often all you need:

```typescript
// db.ts — module-level singleton via Node.js module cache
import { PrismaClient } from '@prisma/client';

export const db = new PrismaClient();
```

Every file that imports `db` gets the same `PrismaClient` instance.

## Reasons to Avoid Overusing Singletons

**Hidden global state:** A singleton is global state with a class wrapper. Global state makes the flow of data unpredictable — a function's behavior depends on the singleton's current state, which was set elsewhere.

**Testing difficulty:** Tests that rely on singletons share state between test cases unless the singleton is explicitly reset. Shared state between tests causes order-dependent test failures — the hardest kind to diagnose.

**Tight coupling:** Code that calls `getDbPool()` directly cannot be tested with a different pool. Code that receives the pool as a parameter can.

## Safer Alternative: Dependency Injection

Instead of reaching for a global singleton, accept the dependency as a parameter:

```typescript
// Testable — accepts the dependency
async function getUserById(
  userId: string,
  dbPool: Pool = getDbPool()  // default uses the singleton, tests can pass a mock
): Promise<User | null> {
  const result = await dbPool.query('SELECT * FROM users WHERE id = $1', [userId]);
  return result.rows[0] ?? null;
}
```

Or at the service/class level, accept dependencies in the constructor rather than importing singletons directly.

## Related Documents

- [Patterns overview](../overview.md)
- [Factory pattern](factory.md)
- [Database choosing](../../../data/choosing.md)
