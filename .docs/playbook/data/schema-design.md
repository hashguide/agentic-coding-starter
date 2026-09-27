---
kind: playbook
domain: data
topics: [data, schema, design]
authority: guidance
---

# Schema Design

Schema design decisions made at the start of a project are expensive to reverse later. The conventions below — standard columns, Row-Level Security, and the soft-delete/hard-delete rule — should be applied from the first migration.

## Standard Columns

Every table should include these columns:

```sql
CREATE TABLE invoices (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  created_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
  deleted_at  TIMESTAMPTZ,          -- NULL = active; non-NULL = soft-deleted

  -- Table-specific columns below
  user_id     UUID NOT NULL REFERENCES users(id),
  client_id   UUID REFERENCES clients(id),
  status      TEXT NOT NULL DEFAULT 'draft',
  total       NUMERIC(10, 2) NOT NULL DEFAULT 0
);

-- Trigger to auto-update updated_at
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER invoices_updated_at
  BEFORE UPDATE ON invoices
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

**Why UUIDs instead of serial integers:** UUIDs are safe to expose in URLs (no sequential enumeration attack), can be generated client-side before the DB write, and don't reveal record counts.

## Row-Level Security (RLS)

RLS policies enforce data access at the database level — a query that doesn't include the correct `WHERE` clause is rejected by the database itself, not just filtered out by application code.

```sql
-- Enable RLS on the invoices table
ALTER TABLE invoices ENABLE ROW LEVEL SECURITY;

-- Policy: users can only see their own invoices
CREATE POLICY invoices_user_isolation ON invoices
  USING (user_id = current_setting('app.current_user_id')::uuid);

-- Set the context at the start of each request (in application code)
await db.$executeRaw`SELECT set_config('app.current_user_id', ${userId}, true)`;
```

RLS is the safety net for multi-tenant data access. Even if application code accidentally omits a `WHERE user_id = ?` clause, RLS prevents the query from returning other users' data.

## The Soft Delete Default

**Default behavior for all user-owned content:** Set `deleted_at` to the current timestamp. The record remains in the database; queries filter it out by default.

```typescript
// Soft delete — sets deleted_at, record stays in DB
await db.invoice.update({
  where: { id: invoiceId, userId },
  data: { deletedAt: new Date() },
});

// All queries must filter soft-deleted records
await db.invoice.findMany({
  where: { userId, deletedAt: null },
});
```

**Benefits of soft delete:** Accidental deletion is recoverable. Audit history is preserved. Foreign key references remain valid. Cascades can be reviewed before finalizing.

## The Hard Delete Exception — Explicit Erasure Requests

**When a user explicitly invokes their right to erasure — via the settings page, a GDPR Article 17 request, or equivalent — a soft-deleted record is legally insufficient.** The user's PII must be permanently removed or fully anonymized.

A soft-deleted record with `deleted_at` set still contains the user's full name, email, and all other PII. It is accessible to database administrators, visible in database dumps, and included in backups. It does not satisfy a legal erasure request.

**Required erasure process:** A background job that:
1. Hard-deletes or anonymizes all PII fields in all tables (`users`, `clients`, session data, audit logs)
2. Cancels any active Stripe subscription
3. Sends a deletion confirmation email
4. Destroys all active sessions for the user

```typescript
// Anonymize user PII — hard delete is also acceptable if no FK constraints prevent it
await db.user.update({
  where: { id: userId },
  data: {
    email: `deleted-${userId}@deleted.invalid`,
    name: '[Deleted User]',
    passwordHash: null,
    deletedAt: new Date(),
  },
});

// Delete associated PII records that don't need to be preserved
await db.session.deleteMany({ where: { userId } });
```

This rule must be applied consistently in: [security/data-privacy.md](../security/data-privacy.md), [compliance/gdpr.md](../compliance/gdpr.md), [design/layouts/settings.md](../experience/layouts/settings.md), and [how-to/setup-data-export-and-deletion.md](../guides/setup-data-export-and-deletion.md).

## Security

A soft-deleted record still contains full PII and does NOT satisfy a legal erasure request. A `deleted_at` timestamp is a display filter, not a deletion. Erasure under GDPR Article 17 requires permanent removal or complete anonymization of all PII fields. See [security/data-privacy.md](../security/data-privacy.md) for the full privacy implementation.

**Encryption at rest for sensitive columns:** Fields containing SSNs, payment method details (not card numbers — those never touch your server), or other highly sensitive data should use field-level encryption in addition to database-level encryption. See [database/encryption.md](encryption.md).

## Related Documents

- [Database ORM](orm.md)
- [Database migrations](migrations.md)
- [Security data privacy](../security/data-privacy.md)
- [Compliance GDPR](../compliance/gdpr.md)
- [How-to: data export and deletion](../guides/setup-data-export-and-deletion.md)
