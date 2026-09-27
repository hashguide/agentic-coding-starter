---
kind: playbook
domain: backend
topics: [backend, runtime, logging]
authority: guidance
---

# Logging

Logs are the primary debugging tool for production issues. They are also a security risk: logs that contain passwords, tokens, or PII give an attacker who gains log access everything they need. Both requirements — comprehensive debugging information and strict data exclusion — must be met simultaneously.

## Log Levels

| Level | When to use | Examples |
|---|---|---|
| `debug` | Verbose development information. Never in production. | SQL query text, full request/response bodies, internal state snapshots |
| `info` | Normal operations worth noting. | Server started, user logged in, invoice created, webhook received |
| `warn` | Something unexpected happened but the request succeeded. | Deprecated API usage, retry succeeded after failure, config fallback used |
| `error` | A request failed or an operation could not complete. | Unhandled exception, external API call failed permanently, DB connection lost |

Production log level: `info` and above. Never `debug` in production — debug logs frequently contain PII, query parameters, and data that should not be persisted.

## Structured Logging Format

Logs should be machine-parseable JSON. Free-text logs are not searchable, cannot be filtered reliably, and cannot be aggregated. Use a structured logging library (pino is the recommended choice for Node.js — fastest JSON logger available).

```typescript
// src/lib/logger.ts
import pino from 'pino';
import { env } from './env';

export const logger = pino({
  level: env.NODE_ENV === 'production' ? 'info' : 'debug',
  redact: {
    paths: [
      'password',
      'passwordHash',
      'token',
      'secret',
      '*.token',
      '*.password',
      'authorization',
      'cookie',
      'req.headers.authorization',
      'req.headers.cookie',
    ],
    censor: '[REDACTED]',
  },
  base: {
    env: env.NODE_ENV,
    version: process.env.npm_package_version,
  },
});
```

**Required fields in every log entry:**

```typescript
// Example structured log entry
{
  "level": "info",
  "time": "2026-06-30T12:00:00.000Z",
  "env": "production",
  "requestId": "req-abc123",  // Correlates all logs for a single request
  "userId": "user-xyz789",    // Which user's session (null for unauthenticated)
  "msg": "invoice.created",
  "invoiceId": "inv-123",
  "total": 250.00
}
```

## Request ID Correlation

Attach a unique request ID to every log entry from a request. This lets you find all logs for a specific failing request in a sea of concurrent requests.

```typescript
// Middleware — attach requestId to every request
import { randomUUID } from 'crypto';

app.use((req, res, next) => {
  req.requestId = randomUUID();
  req.logger = logger.child({ requestId: req.requestId });
  res.setHeader('X-Request-Id', req.requestId); // Return to client for support
  next();
});

// In a route handler
app.get('/api/invoices', requireAuth, async (req, res) => {
  req.logger.info({ userId: req.userId }, 'invoices.list.started');
  const invoices = await invoiceRepo.findAllForUser(req.userId!);
  req.logger.info({ count: invoices.length }, 'invoices.list.completed');
  res.json(invoices);
});
```

## Security

**Never log:** passwords, password hashes, session tokens, API keys, Stripe secret keys, webhook secrets, refresh tokens, full credit card numbers, raw session cookies, or authorization header values.

Pino's `redact` option (shown above) automatically replaces known-sensitive field names with `[REDACTED]`. But it's not a complete safety net — if a secret is passed in an unexpected field name, it won't be caught. The primary defense is not logging sensitive data in the first place.

**Never log full request bodies.** Request bodies may contain passwords (login requests), PII (profile update requests), or payment information. Log request metadata (path, method, user ID) but not the body.

**Log ships are attack surface.** Logs sent to a third-party service (Datadog, Logtail, Papertrail) travel over the network. Ensure the log shipper uses TLS and that the destination service is included in your data processing agreements.

## Related Documents

- [Backend error handling](error-handling.md)
- [Observability logging](../../observability/logging.md)
- [Environments logging by environment](../../delivery/environments/logging-by-environment.md)
- [Security secrets](../../security/secrets.md)
