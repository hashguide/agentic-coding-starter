---
kind: playbook
domain: backend
topics: [backend, runtime, database, optimization]
authority: guidance
---

# Database Optimization

Database performance problems follow predictable patterns: N+1 queries that multiply database round-trips, missing indexes that produce full table scans, connection pools that exhaust under moderate load. Understanding and preventing these patterns is more impactful than any hardware upgrade.

## The N+1 Problem and Its Fix

N+1 occurs when code loads a list of records (1 query) and then loads a related record for each item in the list (N queries). The result is N+1 total queries for what should be 2.

**N+1 example:**

```typescript
// BAD: 1 query for invoices + 1 query per invoice for client = N+1
const invoices = await db.invoice.findMany({ where: { userId } });
// For each of the 100 invoices, this fires a separate query:
const invoicesWithClients = await Promise.all(
  invoices.map(async (invoice) => ({
    ...invoice,
    client: await db.client.findUnique({ where: { id: invoice.clientId } }),
  })),
);
```

**Fix — eager loading (include):**

```typescript
// GOOD: 2 queries total — one for invoices, one JOIN for clients
const invoicesWithClients = await db.invoice.findMany({
  where: { userId },
  include: { client: true },
});
```

**Fix — explicit batching (when include is not available):**

```typescript
// Manual batching with a WHERE IN query
const invoices = await db.invoice.findMany({ where: { userId } });
const clientIds = [...new Set(invoices.map((i) => i.clientId))];
const clients = await db.client.findMany({ where: { id: { in: clientIds } } });
const clientMap = new Map(clients.map((c) => [c.id, c]));

const invoicesWithClients = invoices.map((invoice) => ({
  ...invoice,
  client: clientMap.get(invoice.clientId),
}));
```

Prisma's `include` uses a batch strategy automatically — it doesn't always produce a single JOIN. Use the Prisma query log in development to see actual query counts.

## Indexing Strategy

**Index foreign keys.** Every foreign key column should have an index. Without it, every query joining on that column performs a full table scan.

```sql
-- Create index on foreign key columns
CREATE INDEX idx_invoices_user_id ON invoices(user_id);
CREATE INDEX idx_invoices_client_id ON invoices(client_id);
CREATE INDEX idx_line_items_invoice_id ON line_items(invoice_id);
```

**Index frequently filtered or sorted columns.**

```sql
-- If you frequently query WHERE status = 'pending' ORDER BY due_date
CREATE INDEX idx_invoices_status_due_date ON invoices(status, due_date);

-- Composite index — order matters. Left-most columns are used for filtering.
-- This index benefits: WHERE status = ? | WHERE status = ? AND due_date < ?
-- But NOT: WHERE due_date < ? alone (due_date is not the left-most column)
```

**Index for soft delete queries.** If all queries exclude soft-deleted records (`WHERE deleted_at IS NULL`), a partial index is more efficient:

```sql
CREATE INDEX idx_invoices_active ON invoices(user_id, status)
  WHERE deleted_at IS NULL;
```

**Don't over-index.** Every index adds write overhead (maintained on every INSERT/UPDATE/DELETE). Index only columns that appear in WHERE, JOIN ON, or ORDER BY clauses for high-frequency queries.

## Connection Pooling

Database connections are expensive to open. A connection pool maintains a set of pre-opened connections, handing them out to requests and returning them when done.

```typescript
// Prisma manages connection pooling automatically
// Configure via DATABASE_URL connection limit parameter:
// postgresql://user:pass@host:5432/db?connection_limit=10

// For direct pg usage (without ORM):
import { Pool } from 'pg';

export const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 10,                   // Maximum concurrent connections
  min: 2,                    // Maintain at least 2 connections
  idleTimeoutMillis: 30_000, // Close connections idle for 30s
  connectionTimeoutMillis: 2_000, // Fail if no connection available in 2s
});
```

**Sizing the pool:** A common starting point is `(2 * CPU cores) + effective_spindle_count`. For a typical 2-core VPS with SSD storage: 5–10 connections is a reasonable starting point. The right size depends on your query mix and database capacity.

## Security

All database access goes through the ORM's query builder or parameterized methods. Raw SQL string queries bypass parameterization and are SQL injection vectors.

```typescript
// FORBIDDEN: raw SQL string with string interpolation
const user = await db.$queryRaw(`SELECT * FROM users WHERE email = '${email}'`);
// An attacker can set email to: ' OR '1'='1

// CORRECT: parameterized — ORM handles escaping
const user = await db.user.findUnique({ where: { email } });

// If raw SQL is genuinely needed, use parameterized raw queries:
const user = await db.$queryRaw<User[]>`SELECT * FROM users WHERE email = ${email}`;
// Template literal syntax — Prisma parameterizes the interpolated values
```

## Related Documents

- [Database ORM](../../data/orm.md)
- [Database caching](../../data/caching.md)
- [Database schema design](../../data/schema-design.md)
