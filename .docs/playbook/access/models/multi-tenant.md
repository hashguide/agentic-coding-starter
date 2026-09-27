---
kind: playbook
domain: access
topics: [access, models, multi, tenant]
authority: guidance
---

# Multi-Tenant Architecture

Multi-tenancy means multiple customers share the same application infrastructure. The tenancy model determines how data is isolated between customers — a fundamental security and operational decision that is expensive to change after launch.

## Tenancy Models

### Shared Schema (Single Database, All Tenants)

All tenants' data lives in the same database tables, distinguished by a `tenant_id` (or `organization_id`) column on every tenant-specific table.

```sql
CREATE TABLE invoices (
  id          UUID PRIMARY KEY,
  tenant_id   UUID NOT NULL REFERENCES organizations(id),
  user_id     UUID NOT NULL REFERENCES users(id),
  -- ...
);

-- RLS policy: tenants can only see their own data
ALTER TABLE invoices ENABLE ROW LEVEL SECURITY;
CREATE POLICY invoices_tenant_isolation ON invoices
  USING (tenant_id = current_setting('app.current_tenant_id')::uuid);
```

**Tradeoffs:**

| Dimension | Shared Schema |
|---|---|
| Isolation strength | Moderate — application-level + RLS; a bug could leak data |
| Migration complexity | Low — one migration applies to all tenants |
| Operational cost | Lowest — one database instance |
| Appropriate scale | Most SaaS products |

### Schema-per-Tenant

Each tenant gets their own Postgres schema within the same database instance.

**Tradeoffs:**

| Dimension | Schema-per-Tenant |
|---|---|
| Isolation strength | Strong — each tenant's tables are separate |
| Migration complexity | High — must apply migrations to each schema |
| Operational cost | Medium — one DB, many schemas |
| Appropriate scale | Regulated industries; tenants with strict isolation requirements |

### Database-per-Tenant

Each tenant gets their own database instance.

**Tradeoffs:**

| Dimension | Database-per-Tenant |
|---|---|
| Isolation strength | Maximum — complete database-level isolation |
| Migration complexity | Very high — must coordinate migrations across databases |
| Operational cost | Highest — one database per tenant |
| Appropriate scale | Enterprise contracts requiring guaranteed isolation; compliance-mandated |

## Shared-Schema RLS Code Example

```typescript
// Set tenant context at the start of every request
async function setTenantContext(tenantId: string): Promise<void> {
  await db.$executeRaw`SELECT set_config('app.current_tenant_id', ${tenantId}, true)`;
}

// Middleware — attach tenant context before any DB operations
app.use(requireAuth, async (req, res, next) => {
  const tenantId = await resolveTenantId(req.userId!);
  await setTenantContext(tenantId);
  req.tenantId = tenantId;
  next();
});

// All queries automatically filtered by RLS — no manual WHERE tenant_id needed
const invoices = await db.invoice.findMany({ where: { userId: req.userId } });
// RLS adds: AND tenant_id = current_setting('app.current_tenant_id')
```

## Security

**A single missing WHERE tenant_id clause is a cross-tenant data leak.** Without RLS, application code must include `WHERE tenant_id = ?` on every query that accesses tenant-scoped data. One missed clause exposes all tenants' data.

Row-Level Security at the database level is the mandatory safety net — not an optional extra. Even with application-level tenant filtering, RLS is the last line of defense. A bug in application code that drops the tenant filter is stopped by RLS; without RLS, there is no safety net.

## Related Documents

- [Access model overview](overview.md)
- [Single tenant](single-tenant.md)
- [Database schema design](../../data/schema-design.md)
- [Auth RBAC](../authentication/rbac.md)
