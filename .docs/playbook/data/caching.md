---
kind: playbook
domain: data
topics: [data, caching]
authority: guidance
---

# Database Caching

Database caching reduces load on the primary database by storing query results, pre-computed aggregations, or commonly-accessed records in faster storage. The four main mechanisms are: application-level caching (Redis), read replicas, materialized views, and query result memoization.

## When to Cache at Which Level

| Cache level | When appropriate | Invalidation complexity |
|---|---|---|
| Application-level (Redis) | Data shared across users; data that needs explicit invalidation | Manual — must invalidate on write |
| Query memoization (in-process) | Per-request deduplication; same data requested multiple times in one request | Automatic — cleared per request |
| Read replica | Read-heavy workloads that can tolerate replication lag | None — eventually consistent |
| Materialized view | Expensive aggregations that don't need real-time accuracy | Scheduled refresh |

## Application-Level Caching (Redis)

```typescript
import { createClient } from 'redis';

const redis = createClient({ url: process.env.REDIS_URL });
await redis.connect();

// Cache-aside pattern — check cache, then DB if miss
async function getSubscriptionStatus(userId: string): Promise<SubscriptionStatus> {
  const cached = await redis.get(`subscription:${userId}`);
  if (cached) return JSON.parse(cached) as SubscriptionStatus;

  const subscription = await db.subscription.findUnique({
    where: { userId },
    select: { status: true, planId: true, currentPeriodEnd: true },
  });

  const status = subscription ?? { status: 'free', planId: null, currentPeriodEnd: null };
  await redis.setEx(`subscription:${userId}`, 300, JSON.stringify(status)); // 5-minute TTL
  return status;
}

// Explicit invalidation when subscription changes
async function invalidateSubscriptionCache(userId: string): Promise<void> {
  await redis.del(`subscription:${userId}`);
}
```

**Call `invalidateSubscriptionCache` in:**
- The Stripe webhook handler for subscription updates
- Any admin override that changes plan access

## Read Replicas

Postgres supports streaming replication — a primary write server replicates data to one or more read-only replicas. Read replicas absorb read traffic, freeing the primary for writes.

**Use cases:**
- Heavy reporting queries that take seconds to complete
- Admin dashboards that run complex aggregations
- Full-text search queries that scan large tables

**Replication lag:** Replicas are eventually consistent — a write to the primary may not appear on a replica for 100ms–1000ms. Never use a read replica for queries that must immediately reflect a recent write (e.g., "check if the payment was recorded" immediately after a payment webhook).

```typescript
// Prisma read replica configuration
const db = new PrismaClient({
  datasources: {
    db: {
      url: process.env.DATABASE_URL, // Primary
    },
  },
});

// Use @prisma/extension-read-replicas for automatic replica routing
import { readReplicas } from '@prisma/extension-read-replicas';

const dbWithReplica = db.$extends(
  readReplicas({ url: process.env.DATABASE_READ_REPLICA_URL }),
);
// Read operations automatically route to the replica
```

## Materialized Views

A materialized view is a stored query result that is computed on a schedule and stored as a table. Queries against the materialized view are fast (it's just a table scan); the underlying computation runs on a schedule rather than on every query.

```sql
-- Create a materialized view for monthly invoice summaries
CREATE MATERIALIZED VIEW monthly_invoice_summary AS
  SELECT
    user_id,
    DATE_TRUNC('month', created_at) AS month,
    COUNT(*) AS invoice_count,
    SUM(total) AS total_amount,
    COUNT(*) FILTER (WHERE status = 'paid') AS paid_count
  FROM invoices
  WHERE deleted_at IS NULL
  GROUP BY user_id, DATE_TRUNC('month', created_at);

-- Create an index on the materialized view for fast lookups
CREATE INDEX idx_monthly_summary_user_id ON monthly_invoice_summary(user_id);

-- Refresh on a schedule (concurrently = no table lock during refresh)
REFRESH MATERIALIZED VIEW CONCURRENTLY monthly_invoice_summary;
```

Schedule the refresh via a cron job or a scheduled background worker. Concurrent refresh requires a unique index on the view.

## Related Documents

- [Backend caching](../backend/architecture/caching.md)
- [Database optimization](../backend/runtime/database-optimization.md)
- [Workers scheduled jobs](../backend/async/scheduled-jobs.md)
