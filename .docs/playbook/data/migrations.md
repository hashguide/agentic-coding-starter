---
kind: playbook
domain: data
topics: [data, migrations]
authority: guidance
---

# Database Migrations

A migration is a versioned, repeatable script that transitions the database schema from one state to another. Migrations are code — they live in version control, they are reviewed in pull requests, and they are tested before deployment. Manual `ALTER TABLE` commands in production are not migrations; they are untracked, unreviewable, and irreversible by the application's normal processes.

## The Rule: Schema Changes Always Ship as Migration Files

Every schema change — adding a column, creating a table, adding an index, changing a constraint — must be expressed as a migration file committed to the repository. This ensures:
- Every environment (local, staging, production) is brought to the same schema state by running the same migration files
- A new developer or server can reproduce the full schema by running all migrations in order
- Rollbacks are possible — there is a file that represents the inverse operation

**Prisma migration example:**

```bash
# Make changes to schema.prisma, then generate a migration
npx prisma migrate dev --name add_client_email_verified

# Creates: prisma/migrations/20260630120000_add_client_email_verified/migration.sql
```

The generated SQL is reviewed in the PR alongside the schema change. The migration file is committed.

## The Expand-Contract Pattern (Zero-Downtime Column Removal)

Removing or renaming a column while the application is running requires a multi-step process. If you drop a column in one deploy, the old application code (still running during the deploy window) will error when it tries to access the missing column.

**Step 1 — Expand (deploy alone):** Add the new column. Application continues reading the old column.

```sql
ALTER TABLE invoices ADD COLUMN client_email VARCHAR(255);
```

**Step 2 — Migrate data (can be part of step 1 or a separate job):**

```sql
UPDATE invoices SET client_email = (
  SELECT email FROM clients WHERE clients.id = invoices.client_id
);
```

**Step 3 — Update application code (deploy alone):** Write to both old and new columns. Read from the new column.

**Step 4 — Contract (deploy alone):** Drop the old column. All application instances are now reading from the new column.

```sql
ALTER TABLE invoices DROP COLUMN old_client_email;
```

This pattern adds deployment steps but eliminates downtime and rollback risk.

## Testing Migrations Against Production-Sized Data

Migrations that run instantly on a 100-row development database can take hours on a 10-million-row production database. `ALTER TABLE ... ADD COLUMN` is a table rewrite in some versions of Postgres. Adding a NOT NULL column without a default locks the table.

Before deploying a migration to production:
1. Apply the migration against a staging database with representative data volume
2. Measure the migration duration
3. If duration exceeds ~30 seconds, investigate zero-downtime alternatives:
   - Use `ADD COLUMN ... DEFAULT NULL` (instant) instead of NOT NULL (table rewrite)
   - Build indexes `CONCURRENTLY` (does not lock the table)
   - Use a background migration for large data backfills

```sql
-- SLOW (table lock, row rewrite on large tables)
ALTER TABLE invoices ADD COLUMN archived BOOLEAN NOT NULL DEFAULT FALSE;

-- FAST (no table rewrite — NULL is stored in page metadata)
ALTER TABLE invoices ADD COLUMN archived BOOLEAN;
-- Backfill in batches, then add NOT NULL constraint separately
```

## Related Documents

- [Database schema design](schema-design.md)
- [Database ORM](orm.md)
- [CI/CD pipeline](../delivery/infrastructure/ci-cd.md)
