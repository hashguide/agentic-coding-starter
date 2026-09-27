---
kind: playbook
domain: guides
topics: [guides, test, workers, locally]
authority: guidance
---

# How to Test Workers Locally

**What you'll have at the end:** Local background job queue processing verified, including a forced-failure/retry test.

---

## Step 1 — Start Redis Locally via Docker

BullMQ requires Redis. Run it locally without installing Redis directly:

```bash
docker run -d \
  --name redis-local \
  -p 6379:6379 \
  redis:7-alpine

# Verify Redis is running
docker exec redis-local redis-cli ping
# Should output: PONG
```

---

## Step 2 — Configure Local Environment

```
# .env.local
REDIS_URL=redis://localhost:6379
```

---

## Step 3 — Start Your Worker Process

In one terminal, start the worker process:

```bash
npm run dev:workers
# Or if you run workers with a separate script:
npx ts-node src/workers/index.ts
```

The worker should log that it's connected to Redis and listening for jobs.

---

## Step 4 — Enqueue a Test Job

In a second terminal (or via an API request), enqueue a test job:

```typescript
// scripts/test-enqueue.ts
import { emailQueue } from '../src/workers/queues';

async function main() {
  const job = await emailQueue.add('send-welcome', {
    userId: 'test-user-id',
    email: 'test@example.com',
  });
  console.log('Enqueued job:', job.id);
}

main().then(() => process.exit(0));
```

```bash
npx ts-node scripts/test-enqueue.ts
```

**Verify:** The worker terminal shows the job being picked up and processed.

---

## Step 5 — Verify Job Processing

Check the worker logs and any side effects:

```bash
# Check if the expected side effect occurred
# For an email job, check the Resend/SendGrid sandbox or test inbox
# For a database update, query the DB

# BullMQ dashboard (optional but useful)
npx bull-board
# Opens a web UI showing queues, jobs, and their states
```

---

## Step 6 — Test Failure and Retry

Temporarily modify a job handler to throw on the first attempt:

```typescript
// Temporarily in your worker handler
let attempts = 0;

emailQueue.process('send-welcome', async (job) => {
  attempts++;
  if (attempts < 2) {
    throw new Error('Simulated failure');
  }
  // Normal processing...
});
```

Enqueue a job and observe:
1. The job fails on first attempt
2. The worker waits for the backoff delay (check your config)
3. The job retries and succeeds on second attempt
4. The job shows as "completed" in the queue

```bash
# With BullMQ, check retry behavior
# The job should show attemptsMade = 2 after a retry
```

---

## Step 7 — Test Permanent Failure (DLQ)

Test that a job that fails all retries moves to the dead-letter queue:

```typescript
// Force a job to fail all retries
emailQueue.process('send-welcome', async (job) => {
  throw new Error('Always fails');
});
```

After all retries exhausted:
1. Job appears in the "failed" state in BullMQ
2. Your DLQ handler (if configured) fires
3. Verify an alert or log entry is created — don't silently discard failed jobs

---

## Cleanup

```bash
# Stop the local Redis container when done
docker stop redis-local
docker rm redis-local

# Or keep it running across sessions
docker start redis-local
```

---

## Related Documents

- [Workers overview](../backend/async/overview.md)
- [Workers job patterns](../backend/async/job-patterns.md)
- [Integrations Stripe webhooks](../integrations/providers/stripe/webhooks.md)
