---
kind: playbook
domain: backend
topics: [backend, async]
authority: guidance
---

# Background Workers Overview

Some tasks must not run in the request/response cycle. Tasks that are slow (> a few hundred milliseconds), unreliable (dependent on external APIs), or independently schedulable belong in a background worker. Moving these out of request handlers improves response times, reduces timeout risk, and gives you retry and observability capabilities you cannot have in a synchronous handler.

## What Belongs in a Worker

| Task type | Reasoning |
|---|---|
| **Sending emails** | Transactional email providers can be slow or temporarily unavailable; failures should retry, not block the user's request |
| **AI / LLM calls** | Can take seconds or minutes; streaming responses need dedicated infrastructure |
| **File processing** | Image resizing, PDF generation, archive assembly — CPU and I/O bound, unbounded duration |
| **Third-party API syncs** | External APIs rate-limit; syncs may take multiple requests; failures should not cascade into the user's session |
| **Webhook processing** | Must return 200 immediately to the webhook sender; actual processing happens async |
| **Scheduled reports / aggregations** | Time-triggered, not user-triggered; no request context to attach to |
| **Account deletion and data export** | Involves multiple tables, external API calls (Stripe), and file I/O — far too slow for a synchronous handler |
| **Notification fan-out** | Sending a notification to many recipients must be queued and processed in batches |

**Rule of thumb:** if the task can fail independently of the user's action, or takes longer than the user should wait, it belongs in a worker.

## Queue Technology Options

| Option | Best for | Trade-offs |
|---|---|---|
| **BullMQ + Redis** | High-throughput queues, complex job dependencies, rate limiting | Requires a Redis instance; battle-tested; rich feature set (priorities, delays, repeatable jobs) |
| **pg-boss + Postgres** | Projects already using Postgres; prefer not to add Redis | Slightly lower throughput than Redis-backed queues; simpler operational profile; jobs are durable by default |
| **Inngest** | Serverless deployments; complex multi-step workflows | Managed, no infrastructure; generous free tier; event-driven model; not self-hostable |
| **Trigger.dev** | Complex background jobs with long runtimes and human-readable logs | Managed; strong DX; self-hostable; best for long-running AI or pipeline jobs |

**Recommended default:** BullMQ with Redis for self-hosted deployments. pg-boss if you want zero new infrastructure and already run Postgres. Inngest or Trigger.dev if you're on serverless (Vercel, Cloudflare Workers) where persistent Redis connections don't fit.

## Basic BullMQ Setup

```typescript
import { Queue, Worker } from 'bullmq';
import { connection } from '../redis/client';

// Define queues
export const emailQueue = new Queue('email', { connection });
export const exportQueue = new Queue('exports', { connection });

// Add a job
await emailQueue.add('sendWelcome', { userId: user.id }, {
  attempts: 3,
  backoff: { type: 'exponential', delay: 1000 },
  removeOnComplete: true,
  removeOnFail: 1000, // Keep last 1000 failed jobs
});

// Worker (runs in a separate process)
const emailWorker = new Worker('email', async (job) => {
  if (job.name === 'sendWelcome') {
    const user = await db.user.findUniqueOrThrow({ where: { id: job.data.userId } });
    await emailService.sendWelcome(user);
  }
}, { connection });
```

## Related Documents

- [Job patterns (retry, idempotency, DLQ)](job-patterns.md)
- [Webhook handling](webhooks.md)
- [Scheduled jobs](scheduled-jobs.md)
- [Backend concurrency](../runtime/concurrency/overview.md)
