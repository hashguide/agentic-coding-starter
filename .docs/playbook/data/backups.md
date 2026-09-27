---
kind: playbook
domain: data
topics: [data, backups]
authority: guidance
---

# Database Backups

A backup strategy that has never been tested is not a backup strategy — it's an assumption. The goal of this document is not just to configure backups but to ensure they are demonstrably restorable before a crisis makes testing them urgent.

## Automated Backup Frequency

| Environment | Backup frequency | Retention |
|---|---|---|
| Production | Continuous WAL archiving or daily snapshots at minimum | 30 days |
| Staging | Daily | 7 days |
| Local | Not required (no production data) | — |

**Recommended baseline for a small product:** Daily automated backups with 30-day retention, plus point-in-time recovery (PITR) if supported by your managed database provider.

Managed Postgres providers (Neon, Supabase, RDS, Railway) typically handle automated backups and PITR automatically. Verify what your provider guarantees and where backups are stored (in the same region? Different region?).

For self-hosted Postgres on a VPS, use `pg_dump` with a daily cron job and ship to a separate storage location (S3, Backblaze B2):

```bash
#!/bin/bash
# /opt/scripts/backup-db.sh — run daily via cron
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="/tmp/backup_${TIMESTAMP}.sql.gz"

pg_dump "$DATABASE_URL" | gzip > "$BACKUP_FILE"

# Upload to S3 (or equivalent)
aws s3 cp "$BACKUP_FILE" "s3://${BACKUP_BUCKET}/postgres/${TIMESTAMP}.sql.gz"

# Remove local file
rm "$BACKUP_FILE"

echo "Backup completed: ${TIMESTAMP}"
```

Add to crontab: `0 3 * * * /opt/scripts/backup-db.sh >> /var/log/backup.log 2>&1`

## RTO and RPO Definitions

**Recovery Time Objective (RTO):** The maximum acceptable time to restore service after a failure. "We can tolerate up to 2 hours of downtime during a restore."

**Recovery Point Objective (RPO):** The maximum acceptable amount of data loss, measured in time. "We can tolerate losing up to 24 hours of data."

For a small product product at launch, reasonable targets:
- RTO: < 4 hours (you need time to detect, diagnose, and restore)
- RPO: < 24 hours (daily backups cover this; PITR is required for better RPO)

If your product handles financial transactions or user-generated content that cannot be recovered from user input, your RPO requirements may be stricter. Adjust backup frequency and PITR accordingly.

## Monthly Restore Drills

**A backup that has never been restored has unknown reliability.** Automated backup jobs can fail silently. The backup process may be running successfully while the actual restore fails due to version mismatch, corruption, or missing permissions.

Perform a restore drill monthly:

```bash
# 1. Download the most recent backup
aws s3 cp "s3://${BACKUP_BUCKET}/postgres/latest.sql.gz" /tmp/restore_test.sql.gz

# 2. Create a throwaway restore database
createdb restore_test_$(date +%Y%m%d)

# 3. Restore into it
gunzip < /tmp/restore_test.sql.gz | psql restore_test_$(date +%Y%m%d)

# 4. Run a basic sanity check
psql restore_test_$(date +%Y%m%d) -c "SELECT COUNT(*) FROM users;"

# 5. Verify the count matches production (approximately)
# ...

# 6. Drop the restore database
dropdb restore_test_$(date +%Y%m%d)
```

Record the result in a monthly ops checklist. A restore drill that fails is a critical finding — fix it immediately, before the real disaster requires it.

## Related Documents

- [Database encryption](encryption.md)
- [Infrastructure VPS monitoring](../delivery/infrastructure/vps/monitoring.md)
- [Ops metrics](../operations/metrics.md)
