---
kind: playbook
domain: data
topics: [data, orm]
authority: guidance
---

# ORM Options

An ORM (Object-Relational Mapper) translates between the application's type system and the database. Beyond the API convenience, ORMs provide parameterized queries — the primary defense against SQL injection. For this reason, the ORM's query builder is not an optional convenience; it is the security boundary for database access.

## Comparison Table

| ORM | Type safety | Migration tooling | Query performance overhead | Learning curve |
|---|---|---|---|---|
| **Prisma** | Excellent — generated types from schema | Excellent — declarative schema, auto-generated migrations | Low–Medium (some overhead from abstraction) | Low |
| **Drizzle** | Excellent — types from schema, no code generation | Good — push or migration modes | Very low — thin layer, close to SQL | Low–Medium |
| **Kysely** | Excellent — query builder with inferred types | None built-in (use a separate migration tool) | Very low — SQL builder | Medium |
| **TypeORM** | Good — decorator-based | Good — auto-migration support | Medium | Medium–High |

## Brief Code Examples

**Prisma:**

```typescript
// schema.prisma defines the data model
// Generated types are available automatically

const invoice = await db.invoice.findFirst({
  where: { id: invoiceId, userId, deletedAt: null },
  include: { lineItems: true, client: true },
});
```

**Drizzle:**

```typescript
// Schema defined in TypeScript — types are inferred
import { invoices, lineItems, clients } from './schema';

const invoice = await db
  .select()
  .from(invoices)
  .leftJoin(lineItems, eq(lineItems.invoiceId, invoices.id))
  .leftJoin(clients, eq(clients.id, invoices.clientId))
  .where(and(eq(invoices.id, invoiceId), eq(invoices.userId, userId), isNull(invoices.deletedAt)))
  .limit(1);
```

**Kysely:**

```typescript
const invoice = await db
  .selectFrom('invoices')
  .selectAll()
  .where('id', '=', invoiceId)
  .where('user_id', '=', userId)
  .where('deleted_at', 'is', null)
  .executeTakeFirst();
```

## When to Choose Each

**Prisma:** Best default for TypeScript SaaS projects. The schema-first approach (`schema.prisma`) is easy to read, migrations are straightforward, and the generated Prisma Client has excellent TypeScript types. The main trade-off is build-time code generation (slightly slower CI) and some abstraction overhead.

**Drizzle:** Choose when you want SQL-close queries with no code generation step. Drizzle's schema is TypeScript, and it infers types without generating code. Better choice if you find Prisma's abstraction limiting or its query performance insufficient for a specific use case.

**Kysely:** Choose when you want a typed SQL builder without a schema abstraction. Good for migrations from raw SQL or for complex queries that don't fit ORM conventions. Requires a separate migration tool.

**TypeORM:** Avoid for new projects. Decorator-based API has ergonomic issues; the type safety is weaker than Prisma or Drizzle; and its active development has slowed.

## Security

Raw SQL string queries bypass parameterization and are SQL injection vectors. The ORM's parameterized methods are not optional — they are the security boundary.

```typescript
// FORBIDDEN — SQL injection vulnerability
const result = await db.$queryRaw(`SELECT * FROM users WHERE email = '${email}'`);

// CORRECT — Prisma's parameterized query builder
const user = await db.user.findUnique({ where: { email } });

// CORRECT — if raw SQL is needed, use the parameterized template literal form
const result = await db.$queryRaw<User[]>`SELECT * FROM users WHERE email = ${email}`;
// Prisma treats interpolated values as parameters, not string concatenation
```

## Related Documents

- [Database choosing](choosing.md)
- [Database schema design](schema-design.md)
- [Database optimization](../backend/runtime/database-optimization.md)
- [Repository pattern](../architecture/patterns/structural/repository.md)
