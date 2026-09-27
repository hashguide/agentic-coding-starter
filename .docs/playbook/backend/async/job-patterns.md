---
kind: playbook
domain: backend
topics: [backend, async, job, patterns]
authority: guidance
---

# Job Patterns

Background jobs fail. External APIs go down, databases time out, third-party services rate-limit. A well-designed job handles failure gracefully: retrying with backoff, avoiding duplicate side effects on retry, and giving up cleanly when retries are exhausted.

## Exponential Backoff Retry

Retrying immediately after failure hammers an already-struggling dependency. Exponential backoff spaces retries out, giving external systems time to recover:

```typescript
import { Queue } from 'bullmq';

const emailQueue = new Queue('email', { connection });

// Job configuration with exponential backoff
await emailQueue.add(
  'sendInvoice',
  { userId: 'usr_123', invoiceId: 'inv_456' },
  {
    attempts: 5,
    backoff: {
      type: 'exponential',
      delay: 2000, // Start at 2s, then 4s, 8s, 16s, 32s
    },
    removeOnComplete: true,
    removeOnFail: 500,
  },
);
```

With `delay: 2000` and 5 attempts:
- Attempt 1: immediate
- Attempt 2: 2 seconds after failure
- Attempt 3: 4 seconds after failure
- Attempt 4: 8 seconds after failure
- Attempt 5: 16 seconds after failure

Add jitter in high-volume scenarios to prevent synchronized retry storms:

```typescript
function exponentialDelayWithJitter(attempt: number, baseDelay = 1000): number {
  const exponential = Math.pow(2, attempt) * baseDelay;
  const jitter = Math.random() * baseDelay;
  return exponential + jitter;
}
```

## Idempotency

A job retried after partial completion must not produce duplicate side effects. Design jobs so that running them twice produces the same result as running them once.

```typescript
async function sendInvoiceJob({ userId, invoiceId }: { userId: string; invoiceId: string }) {
  const invoice = await db.invoice.findUniqueOrThrow({ where: { id: invoiceId } });

  // Guard: already sent — skip silently
  if (invoice.emailedAt) {
    return { skipped: true, reason: 'already_sent' };
  }

  await emailService.sendInvoice({ userId, invoice });

  // Mark as sent atomically with the send
  await db.invoice.update({
    where: { id: invoiceId },
    data: { emailedAt: new Date() },
  });

  return { sent: true };
}
```

**Idempotency key pattern:** For external API calls that may charge money or create records, send an idempotency key derived from stable job data:

```typescript
await stripe.invoices.pay(stripeInvoiceId, {
  idempotencyKey: `pay-invoice-${invoiceId}`,
});
```

Stripe (and most payment APIs) will return the same response for duplicate requests with the same idempotency key, preventing double charges.

## Dead Letter Queue

When a job exhausts all retries, it should not silently disappear — it should land in a dead letter queue (DLQ) where it can be inspected and manually retried after the underlying issue is resolved.

```typescript
import { Worker, UnrecoverableError } from 'bullmq';

const worker = new Worker('email', async (job) => {
  try {
    await processEmailJob(job.data);
  } catch (err) {
    // Unrecoverable errors skip the retry queue entirely
    if (err instanceof EmailAddressInvalidError) {
      throw new UnrecoverableError('Invalid email address — will not retry');
    }
    throw err; // Let BullMQ handle retries for other errors
  }
}, { connection });

// Failed jobs land in BullMQ's failed set — queryable via Bull Board or directly
const failedJobs = await emailQueue.getFailed(0, 100);
// Alert when DLQ exceeds a threshold
if (failedJobs.length > 50) {
  await alertService.send({ message: `Email DLQ has ${failedJobs.length} failed jobs` });
}
```

Set up a monitoring dashboard (Bull Board or equivalent) to inspect and replay failed jobs without code changes.

## Security

**Never include secrets, auth tokens, or PII in job payloads.** Job payloads are stored in Redis or the database, serialized as plain JSON, and visible to anyone with read access to the queue backend. A job payload that contains `{ email: 'user@example.com', password: 'hunter2' }` is a data breach waiting to happen.

Pass only stable record IDs in the payload. Inside the worker, look up the data from the database using a trusted server-side connection:

```typescript
// Safe: job payload contains only an ID
await emailQueue.add('sendWelcome', { userId: user.id });

// Unsafe: job payload contains PII
await emailQueue.add('sendWelcome', { email: user.email, name: user.name });
```

## Related Documents

- [Workers overview](overview.md)
- [Webhook handling](webhooks.md)
- [Scheduled jobs](scheduled-jobs.md)
- [Backend fault tolerance](../runtime/concurrency/fault-tolerance.md)
