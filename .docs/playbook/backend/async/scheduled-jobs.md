---
kind: playbook
domain: backend
topics: [backend, async, scheduled, jobs]
authority: guidance
---

# Scheduled Jobs

Scheduled jobs (cron jobs) run on a time-based trigger rather than a user action. They handle recurring maintenance tasks: purging old data, sending digest emails, aggregating metrics, syncing with external services. Like all background jobs, they must be idempotent and monitored.

## Define in Code, Not Infrastructure

Scheduled job definitions belong in the application codebase, not in a crontab, a CI/CD config, or a cloud scheduler console. Infrastructure-defined cron jobs:
- Are not version-controlled
- Have no audit trail for changes
- Are invisible to code reviewers
- Are lost when infrastructure is rebuilt

With BullMQ:

```typescript
import { Queue } from 'bullmq';

const maintenanceQueue = new Queue('maintenance', { connection });

// Defined once at startup — BullMQ persists the schedule in Redis
await maintenanceQueue.add(
  'retentionPurge',
  {},
  {
    repeat: { pattern: '0 3 * * *' }, // Daily at 3 AM UTC
    jobId: 'retention-purge-daily',   // Prevents duplicate schedule registration
  },
);

await maintenanceQueue.add(
  'digestEmails',
  {},
  {
    repeat: { pattern: '0 9 * * 1' }, // Every Monday at 9 AM UTC
    jobId: 'digest-emails-weekly',
  },
);
```

With pg-boss (Postgres-backed):

```typescript
import PgBoss from 'pg-boss';

const boss = new PgBoss(process.env.DATABASE_URL!);
await boss.start();

await boss.schedule('retention-purge', '0 3 * * *', {});
boss.work('retention-purge', async () => {
  await runRetentionPurge();
});
```

## Idempotency Requirement

A scheduled job must be safe to run twice. Failures, restarts, and clock drift can cause a job to fire more than once. Design every scheduled job so that a duplicate run produces the same outcome as a single run.

```typescript
async function retentionPurge(): Promise<void> {
  const cutoff = new Date(Date.now() - 30 * 24 * 60 * 60 * 1000);

  // Idempotent: deleteMany where condition is deterministic
  // Running twice deletes the same records (already-deleted records are gone)
  await db.project.deleteMany({
    where: { deletedAt: { lte: cutoff } },
  });
}
```

If a job must update a record exactly once, use a status field:

```typescript
async function sendWeeklyDigests(): Promise<void> {
  const cutoff = new Date(Date.now() - 7 * 24 * 60 * 60 * 1000);

  // Only process users who haven't received this week's digest
  const users = await db.user.findMany({
    where: { lastDigestSentAt: { lt: cutoff } },
  });

  for (const user of users) {
    await emailService.sendDigest(user);
    await db.user.update({
      where: { id: user.id },
      data: { lastDigestSentAt: new Date() },
    });
  }
}
```

## Dead Man's Switch (Heartbeat Monitoring)

A scheduled job that silently stops running is often worse than one that fails loudly. If the retention purge stops running for two weeks, you accumulate data you should have deleted. You won't notice unless you're watching.

Use a dead man's switch: the job sends a heartbeat ping to a monitoring service on each successful completion. If the monitoring service doesn't receive a ping within the expected interval, it alerts you.

```typescript
async function retentionPurge(): Promise<void> {
  await runRetentionPurge(); // actual work

  // Heartbeat ping after successful completion
  await fetch(process.env.HEALTHCHECK_URL_RETENTION_PURGE!, { method: 'GET' })
    .catch((err) => console.error('Heartbeat ping failed', err));
  // Do not throw if the ping fails — the purge succeeded; only alerting failed
}
```

Services that provide dead man's switch URLs: Healthchecks.io, Better Uptime, Cronitor. Most have a free tier sufficient for small numbers of scheduled jobs.

## Related Documents

- [Workers overview](overview.md)
- [Job patterns (retry, idempotency)](job-patterns.md)
- [Data retention](../../compliance/data-retention.md)
- [Observability uptime](../../observability/uptime.md)
