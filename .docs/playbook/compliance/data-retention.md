---
kind: playbook
domain: compliance
topics: [compliance, data, retention]
authority: guidance
---

# Data Retention

Retaining data longer than necessary is both a legal liability and an unnecessary security risk. Every category of personal data should have a defined retention period, and deletion should be automated — not dependent on a manual process someone remembers to run.

## Setting Retention Periods

For each data category, answer: how long does this data serve a legitimate purpose? After that point, it should be deleted or anonymized.

| Data category | Retention period | Justification |
|---|---|---|
| Active user account data | Duration of account + 30 days post-deletion | Account may be reactivated within 30 days |
| Payment and invoice records | 7 years | Legal / tax requirement |
| Server access logs | 90 days | Incident investigation; diminishing value after 30 days |
| Application error logs | 90 days | Debugging; errors older than 90 days are not actionable |
| Analytics events | 24 months | Trend analysis; data older than 2 years rarely affects decisions |
| Soft-deleted user content | 30 days | Recovery window; purge after |
| Support ticket content | 3 years | Reference for repeat issues |

Define these periods in a written data map and keep it current. GDPR's principle of storage limitation requires that data is "kept in a form which permits identification of data subjects for no longer than is necessary."

## Automated Deletion Job

Manual deletion is unreliable. Implement a scheduled job that runs nightly or weekly:

```typescript
import { db } from '../database/client';

// Runs nightly — deletes records past their retention window
async function runRetentionPurge(): Promise<void> {
  const now = new Date();

  // Purge soft-deleted user content older than 30 days
  const thirtyDaysAgo = new Date(now.getTime() - 30 * 24 * 60 * 60 * 1000);
  const deletedProjects = await db.project.deleteMany({
    where: { deletedAt: { lte: thirtyDaysAgo } },
  });

  // Purge server logs older than 90 days
  const ninetyDaysAgo = new Date(now.getTime() - 90 * 24 * 60 * 60 * 1000);
  const deletedLogs = await db.serverLog.deleteMany({
    where: { createdAt: { lte: ninetyDaysAgo } },
  });

  // Anonymize analytics events older than 24 months
  const twoYearsAgo = new Date(now.getTime() - 2 * 365 * 24 * 60 * 60 * 1000);
  await db.analyticsEvent.updateMany({
    where: { createdAt: { lte: twoYearsAgo } },
    data: { userId: null, ipAddress: null, userAgent: null },
  });

  console.log({
    message: 'Retention purge complete',
    deletedProjects: deletedProjects.count,
    deletedLogs: deletedLogs.count,
  });
}
```

Monitor this job with a dead man's switch — alert if it stops running. See [workers/scheduled-jobs.md](../backend/async/scheduled-jobs.md) for heartbeat monitoring.

## Audit Trail

Some data cannot be deleted even when a user requests erasure because it is required for legal reasons (billing records, fraud investigation, regulatory compliance). Distinguish between:

- **Operational data:** deleted or anonymized per retention policy and on user request
- **Legal hold data:** retained for the minimum legally required period, then purged; PII in this data is anonymized as much as possible while preserving the legally required facts (e.g., invoice amount, date — but not the customer's email or name)

Document which categories are on legal hold and why, so you can respond accurately to data subject requests.

## Security

Data you retain longer than necessary is attack surface and legal liability. A breach exposing three years of event logs is more damaging than one exposing 90 days. A regulator reviewing your practices after a breach who finds no retention policy is looking at a compounding violation. Automated deletion is both a security control and a compliance control.

## Related Documents

- [Security data privacy](../security/data-privacy.md)
- [GDPR compliance](gdpr.md)
- [Database schema design](../data/schema-design.md)
- [Scheduled jobs](../backend/async/scheduled-jobs.md)
