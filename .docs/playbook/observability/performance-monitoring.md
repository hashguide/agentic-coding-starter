---
kind: playbook
domain: observability
topics: [observability, performance, monitoring]
authority: guidance
---

# Performance Monitoring

Performance monitoring tracks how fast the application responds to real users. The key metric is not average response time — it is the 95th percentile, which represents the worst experience your non-tail users receive.

## APM Tool Options

| Tool | Strengths | Pricing |
|---|---|---|
| **Sentry Performance** | Integrated with error monitoring, traces individual requests | Included in Sentry plans |
| **Datadog APM** | Full observability platform, excellent dashboards, auto-instrumentation | Expensive at scale; overkill for small teams |
| **New Relic** | Mature platform, generous free tier | Complex pricing |
| **OpenTelemetry + self-hosted** | Vendor-neutral, full control | Engineering overhead |

**Recommended:** Sentry Performance for most SaaS products — it's already installed for error monitoring, and adding performance traces requires minimal additional setup.

## p50 vs. p95 vs. p99 — Why Averages Lie

| Percentile | Meaning | Why it matters |
|---|---|---|
| **p50** (median) | 50% of requests are faster than this | Describes the typical experience |
| **p95** | 95% of requests are faster than this | Describes the "bad but not worst" experience — the majority of slow responses |
| **p99** | 99% of requests are faster than this | Describes the tail — frequent enough to affect real users |

**Average response time hides outliers.** If 95% of requests take 100ms and 5% take 10,000ms, the average might be 600ms — which sounds acceptable but misses that 5% of users are waiting 10 seconds.

Optimize for p95, not average. p99 is worth tracking for intermittent issues.

## Latency Targets

| Request type | p50 target | p95 target |
|---|---|---|
| API read (simple DB query) | <50ms | <200ms |
| API write (DB write + validation) | <100ms | <500ms |
| Page render (SSR) | <200ms | <800ms |
| Background job pickup-to-start | <1s | <5s |

These are starting targets for a typical SaaS. Calibrate to your actual workload.

## Distributed Tracing

When a request touches multiple services (app server → database → Redis → external API), distributed tracing shows exactly how long each hop takes. This matters when optimizing slow requests.

**When simple request logging is sufficient:**
- Single-service app with one database
- Most slow requests are attributable to one obvious cause (missing DB index, N+1 query)

**When distributed tracing (OpenTelemetry) becomes necessary:**
- Microservices architecture where a request spans multiple services
- Multiple databases or cache layers
- Slow requests where the bottleneck is not obvious from simple logging

## Enabling Performance Monitoring with Sentry

```typescript
import * as Sentry from '@sentry/node';

Sentry.init({
  dsn: process.env.SENTRY_DSN!,
  tracesSampleRate: 0.1,  // Sample 10% of requests
  // Or use a sampler for more control:
  tracesSampler: (samplingContext) => {
    // Always trace slow requests
    if (samplingContext.request?.url?.includes('/api/critical')) return 1.0;
    // Sample 5% of everything else
    return 0.05;
  },
});
```

## Identifying Slow Queries

Enable query timing in your ORM to surface slow database queries:

```typescript
// Prisma — log slow queries
const db = new PrismaClient({
  log: [
    { level: 'query', emit: 'event' },
  ],
});

db.$on('query', (e) => {
  if (e.duration > 100) {
    logger.warn({ query: e.query, durationMs: e.duration }, 'Slow query detected');
  }
});
```

Alert when a query that typically takes 5ms suddenly takes 500ms — this signals a missing index or lock contention.

## Related Documents

- [Observability logging](logging.md)
- [Observability error monitoring](error-monitoring.md)
- [Observability uptime](uptime.md)
