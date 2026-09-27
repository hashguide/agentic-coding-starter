---
kind: playbook
domain: delivery
topics: [delivery, environments, logging, by, environment]
authority: guidance
---

# Logging by Environment

Log verbosity must be calibrated per environment. Over-logging in production exposes sensitive data and creates noise. Under-logging in development slows debugging.

## Log Level by Environment

| Environment | Log level | Included |
|---|---|---|
| **Local** | `debug` and above | All request/response bodies, SQL query logs, timing data, verbose context — everything useful for debugging |
| **Staging** | `info` and above | Matches production levels for realistic pre-launch verification. No request bodies or PII. |
| **Production** | `info` and above | Business events, warnings, errors. PII-redacted. No request bodies, no sensitive headers. |

Staging deliberately mirrors production log levels. The goal of staging is to validate that production behavior is correct — including the production-equivalent logging behavior.

## Log Level Definitions

| Level | When to use |
|---|---|
| `debug` | Detailed diagnostic information; only useful during development |
| `info` | Normal business events: user signed up, payment processed, job completed |
| `warn` | Unexpected but recoverable states: retry triggered, deprecated API call received |
| `error` | Failures that require attention: job failed, payment declined, database error |

## Implementation

```typescript
// lib/logger.ts
import pino from 'pino';

export const logger = pino({
  level: process.env.LOG_LEVEL ?? (process.env.NODE_ENV === 'development' ? 'debug' : 'info'),
  redact: {
    paths: [
      'req.headers.authorization',
      'req.headers.cookie',
      'body.password',
      'body.token',
      'body.cardNumber',
      'body.cvv',
    ],
    censor: '[REDACTED]',
  },
});
```

## What Must Never Appear in Production Logs

Even at `info` level, these must never appear in any production log entry:

- Passwords or password hashes
- Session tokens or JWT tokens
- Raw API keys or secrets
- Full credit card numbers, CVV, or PAN data
- User PII in query parameters (log request path, not query params)
- Request or response bodies (which may contain any of the above)

```typescript
// Safe: log the event, not the data
logger.info({ userId, action: 'password.change' }, 'User changed password');

// Unsafe: log contains PII and credentials
logger.info({ body: req.body }, 'Received request'); // ← Never do this in production
```

## Local Development: Verbose SQL Logging

In development, enabling SQL query logging helps identify N+1 queries and slow operations:

```typescript
// prisma/client.ts
export const db = new PrismaClient({
  log: process.env.NODE_ENV === 'development'
    ? ['query', 'info', 'warn', 'error']
    : ['warn', 'error'],
});
```

This must never be enabled in production — SQL logs include query parameters which may contain user data.

## Security

Production log verbosity must never include request bodies, query params, or fields that could contain user PII or credentials. Logs are stored, often for months, and are a common target in breach investigations. Data that appears in logs becomes part of the attack surface.

## Related Documents

- [Environments overview](overview.md)
- [Observability logging](../../observability/logging.md)
- [Environments error exposure by environment](error-exposure-by-environment.md)
