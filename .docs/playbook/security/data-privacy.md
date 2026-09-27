---
kind: playbook
domain: security
topics: [security, data, privacy]
authority: guidance
---

# Data Privacy

Data privacy is both a legal requirement and a product quality signal. Users who trust your product with their data expect you to use only what is needed, protect it, and delete it completely when asked. This document covers data minimization, export, and the erasure requirements that govern account deletion.

## Data Minimization

Collect only the data your product actually uses. Every additional field you collect is:
- An additional liability in a breach
- An additional obligation under GDPR, CCPA, and similar regulations
- An additional maintenance burden

Before adding a new field to a user record, confirm: is this field displayed somewhere, used in a calculation, or required by a third-party integration? If the answer is no, do not collect it.

**Retention:** Data should not live longer than it needs to. Define a retention schedule:

| Data type | Retention period | Action after period |
|---|---|---|
| Active user account | Indefinite (while account exists) | — |
| Soft-deleted records | 30 days | Purge via scheduled job |
| Payment records | 7 years | Archive (legal requirement) |
| Server logs | 90 days | Delete |
| Analytics events | 24 months | Delete or anonymize |

## Data Export Endpoint

GDPR Article 20 (right to portability) requires that users can download all data you hold about them. Implement this as an async job — the data may take time to assemble across multiple tables:

```typescript
// POST /api/account/export
app.post('/api/account/export', requireAuth, asyncHandler(async (req, res) => {
  const existing = await db.exportJob.findFirst({
    where: { userId: req.userId, completedAt: null, createdAt: { gte: new Date(Date.now() - 24 * 60 * 60 * 1000) } },
  });

  if (existing) {
    res.json({ status: 'pending', message: 'Export already in progress.' });
    return;
  }

  const job = await db.exportJob.create({ data: { userId: req.userId } });
  await exportQueue.add('assembleExport', { jobId: job.id, userId: req.userId });

  res.json({ status: 'pending', message: 'Export started. You will receive an email when it is ready.' });
}));
```

```typescript
// Worker: assembleExport
async function assembleExport({ jobId, userId }: { jobId: string; userId: string }) {
  const user = await db.user.findUniqueOrThrow({ where: { id: userId } });
  const projects = await db.project.findMany({ where: { userId } });
  const invoices = await db.invoice.findMany({ where: { userId } });

  const archive = JSON.stringify({ user, projects, invoices }, null, 2);
  const url = await storageService.uploadTempFile(`exports/${jobId}.json`, archive, { expiresIn: '48h' });

  await db.exportJob.update({ where: { id: jobId }, data: { completedAt: new Date(), downloadUrl: url } });

  await emailService.sendTransactional({
    to: user.email,
    subject: 'Your data export is ready',
    html: exportReadyEmailTemplate({ downloadUrl: url }),
    text: `Your data export is ready: ${url}`,
  });
}
```

## Data Deletion — Hard Delete Required for Erasure Requests

**A soft-deleted record is NOT sufficient to fulfill a legal erasure request.**

When a user's account is soft-deleted (a `deleted_at` timestamp is set), their PII remains in the database. This preserves it for recovery and audit purposes — which is appropriate for internal operations. But it does not satisfy GDPR Article 17 (right to erasure) or CCPA's right to delete.

When a user explicitly requests deletion of their account or invokes their right to erasure, a background job must:

1. Hard delete or fully anonymize all PII fields across all tables
2. Cancel any active Stripe subscriptions
3. Send a deletion confirmation email
4. Destroy the user's active sessions

```typescript
// Worker: processAccountDeletion
async function processAccountDeletion({ userId }: { userId: string }) {
  const user = await db.user.findUniqueOrThrow({ where: { id: userId } });

  // 1. Cancel active Stripe subscription
  if (user.stripeSubscriptionId) {
    await stripe.subscriptions.cancel(user.stripeSubscriptionId);
  }

  // 2. Hard delete or anonymize all user-owned data
  await db.$transaction([
    // Hard delete rows the user owns
    db.exportJob.deleteMany({ where: { userId } }),
    db.project.deleteMany({ where: { userId } }),

    // Anonymize the user record itself (preserve for billing audit trail)
    db.user.update({
      where: { id: userId },
      data: {
        email: `deleted-${userId}@anonymized.invalid`,
        name: 'Deleted User',
        passwordHash: '',
        stripeCustomerId: null,
        stripeSubscriptionId: null,
        deletedAt: new Date(),
        erasedAt: new Date(),
      },
    }),
  ]);

  // 3. Send confirmation
  await emailService.sendTransactional({
    to: user.email,
    subject: 'Your account has been deleted',
    html: deletionConfirmationEmailTemplate(),
    text: 'Your account and all associated data have been permanently deleted.',
  });

  // 4. Destroy sessions — handled by auth session middleware on next request,
  //    or force-invalidate all sessions in the session store
  await sessionStore.deleteByUserId(userId);
}
```

## Security

**Soft delete preserves PII in the database and does not satisfy a legal erasure request.** A `deleted_at` timestamp hides the record from queries but keeps every PII field intact. If a regulator, data subject, or court asks for evidence that data was deleted, a soft-deleted record with name, email, and payment history still present does not constitute deletion.

Legal erasure requires either hard deletion (removing the row entirely) or full anonymization (replacing all PII fields with non-identifying placeholder values). Anonymization is preferable when the row is needed for referential integrity or billing audit history.

## Related Documents

- [Database schema design (soft/hard delete rule)](../data/schema-design.md)
- [GDPR compliance](../compliance/gdpr.md)
- [Settings layout (export and deletion UI)](../experience/layouts/settings.md)
- [Data export and deletion how-to](../guides/setup-data-export-and-deletion.md)
- [Security checklist](checklist.md)
