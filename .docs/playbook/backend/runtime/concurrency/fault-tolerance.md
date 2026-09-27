---
kind: playbook
domain: backend
topics: [backend, runtime, concurrency, fault, tolerance]
authority: guidance
---

# Fault Tolerance

External dependencies fail. Databases become slow, third-party APIs return 503s, downstream services run out of capacity. Fault tolerance patterns prevent a failure in one dependency from cascading into a full system failure. The goal is not to prevent failures — it's to contain them.

## Circuit Breaker

A circuit breaker monitors calls to an external dependency and stops making calls when the failure rate exceeds a threshold. After a cooling period, it allows a small number of test calls through. If those succeed, the circuit closes (normal operation resumes). If they fail, the circuit stays open (failing fast without making real calls).

```typescript
class CircuitBreaker {
  private failureCount = 0;
  private lastFailureTime: number | null = null;
  private state: 'closed' | 'open' | 'half-open' = 'closed';

  constructor(
    private readonly threshold: number = 5,         // failures before opening
    private readonly resetTimeMs: number = 60_000,  // cooldown period
  ) {}

  async call<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === 'open') {
      if (Date.now() - this.lastFailureTime! > this.resetTimeMs) {
        this.state = 'half-open';
      } else {
        throw new Error('CIRCUIT_OPEN'); // Fail fast — don't attempt the call
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (err) {
      this.onFailure();
      throw err;
    }
  }

  private onSuccess(): void {
    this.failureCount = 0;
    this.state = 'closed';
  }

  private onFailure(): void {
    this.failureCount++;
    this.lastFailureTime = Date.now();
    if (this.failureCount >= this.threshold) {
      this.state = 'open';
    }
  }
}

// Usage
const emailBreaker = new CircuitBreaker(5, 60_000);

async function sendEmail(params: EmailParams): Promise<void> {
  await emailBreaker.call(() => emailProvider.send(params));
}
```

For production, use a library like `opossum` (Node.js) which handles concurrency, metrics, and event emission correctly.

## Bulkhead Pattern

The bulkhead pattern isolates failures by dividing resources into separate pools. When one pool is exhausted (e.g., all database connections are consumed by slow report queries), the other pool (for user-facing requests) remains available.

```typescript
// Separate connection pools for different workload types
const userFacingPool = new Pool({ connectionString: DATABASE_URL, max: 20 });
const backgroundJobPool = new Pool({ connectionString: DATABASE_URL, max: 5 });

// User-facing requests use the large pool
app.get('/api/invoices', async (req, res) => {
  const client = await userFacingPool.connect();
  try {
    const result = await client.query('SELECT * FROM invoices WHERE user_id = $1', [userId]);
    res.json(result.rows);
  } finally {
    client.release();
  }
});

// Background jobs use the separate smaller pool — can't starve user requests
jobWorker.process('generateReport', async (job) => {
  const client = await backgroundJobPool.connect();
  // ... slow report generation
  client.release();
});
```

## Graceful Degradation

When a non-critical dependency is down, serve cached/stale data or a reduced feature set rather than failing entirely.

```typescript
async function getDashboardData(userId: string): Promise<DashboardData> {
  const [invoices, subscriptionStatus] = await Promise.allSettled([
    invoiceRepo.findAllForUser(userId),
    billingService.getSubscriptionStatus(userId),
  ]);

  return {
    invoices: invoices.status === 'fulfilled' ? invoices.value : [],
    subscription: subscriptionStatus.status === 'fulfilled'
      ? subscriptionStatus.value
      : 'unknown', // Degrade gracefully — show unknown rather than error
  };
}
```

**Critical vs. non-critical:** Graceful degradation is for non-critical dependencies. For critical ones (the database, authentication), failing fast is the correct response — there is no meaningful degraded mode.

## Retry Strategies

For transient failures (network blip, momentary API unavailability), retry with exponential backoff:

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  options: { maxAttempts: number; baseDelayMs: number } = { maxAttempts: 3, baseDelayMs: 100 },
): Promise<T> {
  for (let attempt = 1; attempt <= options.maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (err) {
      if (attempt === options.maxAttempts) throw err;
      const delay = options.baseDelayMs * 2 ** (attempt - 1); // Exponential backoff
      await new Promise((resolve) => setTimeout(resolve, delay));
    }
  }
  throw new Error('Unreachable');
}
```

See also [workers/job-patterns.md](../../async/job-patterns.md) for retry strategies in background jobs.

## Related Documents

- [Concurrency overview](overview.md)
- [Workers job patterns](../../async/job-patterns.md)
- [Backend error handling](../error-handling.md)
