---
kind: playbook
domain: observability
topics: [observability, error, monitoring]
authority: guidance
---

# Error Monitoring

Error monitoring captures unhandled exceptions and unexpected states in production, alerting the team immediately when new errors appear or error rates spike.

## Tool Options

| Tool | Strengths | Pricing |
|---|---|---|
| **Sentry** | Industry standard, excellent error grouping, performance traces, session replay | Free tier (5k errors/month), scales with usage |
| **Highlight.io** | Open-source friendly, session replay, logs correlation | Free tier generous, self-hostable |
| **Axiom** | Logs + events + traces in one, great DX, low cost at scale | Free tier, usage-based |

**Recommended for most teams:** Sentry. Widest ecosystem support, best error grouping algorithm, integrations with every major framework.

## What Context to Attach to Every Captured Error

```typescript
import * as Sentry from '@sentry/node';

// At application startup
Sentry.init({
  dsn: process.env.SENTRY_DSN!,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1,  // 10% of requests traced; adjust based on volume
});

// On each authenticated request: set user context
Sentry.setUser({
  id: req.session.userId,
  // Do not include email or other PII unless required and compliant with privacy policy
});

// Attach context to every error
Sentry.setTag('plan', user.subscriptionPlan);
Sentry.setContext('request', {
  path: req.path,
  method: req.method,
  requestId: req.requestId,
});
```

**Required context on every error:**
- User ID (not email — ID is enough to look up details in admin panel)
- Subscription plan (helps prioritize: are paying users affected?)
- Request path and method
- Environment
- Request ID (to correlate with logs)

## Alerting Thresholds

| Alert trigger | Response |
|---|---|
| **New error type** | Alert immediately — any new error type is a new regression |
| **Error rate +50% vs. 1h avg** | Alert within 5 minutes |
| **P95 latency +100% vs. baseline** | Alert within 5 minutes |
| **Specific critical error** (`PAYMENT_FAILED`, `DB_CONNECTION_LOST`) | Alert immediately |

Configure alerts to go to a channel the team checks frequently — Slack, PagerDuty, or SMS. Email-only alerts for production issues are missed.

## Filtering Noise

Not every exception warrants a page. Filter out:
- 404 errors for expected missing resources (bots, old links)
- 400 validation errors (user input problems, not bugs)
- Cancelled requests (user navigated away)

```typescript
Sentry.init({
  // ...
  ignoreErrors: [
    'ResizeObserver loop limit exceeded',
    /^Request aborted$/,
  ],
  beforeSend(event) {
    // Suppress 404 and 400 errors from alerting
    if (event.tags?.statusCode === '404' || event.tags?.statusCode === '400') {
      return null; // Don't send to Sentry
    }
    return event;
  },
});
```

## Express Integration

```typescript
import * as Sentry from '@sentry/node';

// Error handler — must be last middleware, after all routes
app.use(Sentry.expressErrorHandler());

// Custom error handler runs after Sentry captures the error
app.use(errorHandler);
```

## Related Documents

- [Observability logging](logging.md)
- [Observability uptime](uptime.md)
- [Environments error exposure by environment](../delivery/environments/error-exposure-by-environment.md)
