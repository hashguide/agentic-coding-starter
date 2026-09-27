---
kind: playbook
domain: operations
topics: [operations, admin, audit, log]
authority: guidance
---

# Admin Audit Log

The audit log is the accountability layer for every admin action. It must be append-only: no record can be edited or deleted, even by other admins.

## Schema

```sql
-- Prisma schema
model AuditLog {
  id         String   @id @default(cuid())
  actorId    String                      -- Admin user who performed the action
  action     String                      -- Namespaced action key, e.g. "user.suspend"
  targetId   String?                     -- ID of the resource acted on (userId, flagId, etc.)
  targetType String?                     -- "user" | "feature_flag" | "billing" | etc.
  metadata   Json     @default("{}")     -- Before/after state, reason, extra context
  ipAddress  String?
  userAgent  String?
  createdAt  DateTime @default(now())

  // No updatedAt — audit logs are immutable
}
```

**No `updatedAt` column.** The absence of an update timestamp is itself a design signal: these rows are never updated.

## Append-Only Enforcement

Enforce immutability at the database level, not just by convention:

```sql
-- PostgreSQL: revoke UPDATE and DELETE on the audit_logs table for the application user
REVOKE UPDATE, DELETE ON TABLE "AuditLog" FROM app_user;

-- The application DB user can INSERT and SELECT, but never UPDATE or DELETE
GRANT INSERT, SELECT ON TABLE "AuditLog" TO app_user;
```

If using a managed database without row-level permission configuration, use a **separate database user** with insert-only access specifically for writing audit logs. The application's main connection should not have permission to delete audit records.

## Writing Audit Events

```typescript
import { db } from './db';

interface AuditLogEntry {
  actorId: string;
  action: string;
  targetId?: string;
  targetType?: string;
  metadata?: Record<string, unknown>;
  ipAddress?: string;
  userAgent?: string;
}

export async function writeAuditLog(entry: AuditLogEntry): Promise<void> {
  await db.auditLog.create({ data: entry });
}
```

Call `writeAuditLog` immediately after every admin action, in the same request handler, before returning a response.

## Standard Action Namespace

Use a consistent namespacing scheme for action keys:

| Action key | When it fires |
|---|---|
| `user.suspend` | Admin suspends a user account |
| `user.unsuspend` | Admin restores a suspended account |
| `user.delete` | Admin hard-deletes a user |
| `user.role.change` | Admin changes a user's role |
| `user.impersonate.start` | Admin begins impersonating a user |
| `user.impersonate.end` | Impersonation session ended |
| `billing.plan.override` | Admin manually sets a user's plan |
| `billing.refund.issued` | Admin issues a refund |
| `feature_flag.updated` | Admin changes a flag's global enabled/rollout |
| `feature_flag.override.set` | Admin sets a per-user or per-plan flag override |

## Querying the Audit Log

```typescript
// GET /admin/audit-log?actorId=&targetId=&action=&page=1
app.get('/admin/audit-log', requireAdmin, async (req, res) => {
  const { actorId, targetId, action, page = '1' } = req.query as Record<string, string>;
  const pageNum = Math.max(1, parseInt(page));

  const logs = await db.auditLog.findMany({
    where: {
      ...(actorId && { actorId }),
      ...(targetId && { targetId }),
      ...(action && { action: { contains: action } }),
    },
    orderBy: { createdAt: 'desc' },
    skip: (pageNum - 1) * 50,
    take: 50,
  });

  res.json(logs);
});
```

## Retention

Audit logs should be retained for a minimum of 12 months for compliance and incident investigation purposes. Do not apply the same automated deletion job that runs on user-generated content. If storage costs become a concern, archive to cold storage rather than deleting.

## Security

Audit logs that can be edited or deleted provide no meaningful accountability. The schema must enforce append-only writes at the database level.

**Why database-level enforcement?** Application-level "don't delete audit records" conventions fail silently — a bug, a SQL migration script, or a compromised admin account can bypass application logic. Revoking DELETE/UPDATE at the database user level makes it structurally impossible.

## Related Documents

- [Admin overview](overview.md)
- [Admin user management](user-management.md)
- [Compliance GDPR](../../compliance/gdpr.md)
- [Security overview](../../security/overview.md)
