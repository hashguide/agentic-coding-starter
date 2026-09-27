---
kind: playbook
domain: observability
topics: [observability, logging]
authority: guidance
---

# Structured Logging

Structured logging writes log entries as JSON objects rather than plain text strings. This makes logs searchable, filterable, and parseable by log aggregation services.

## Required Fields in Every Log Entry

```json
{
  "level": "info",
  "time": "2025-01-15T14:23:05.123Z",
  "msg": "User completed checkout",
  "userId": "usr_abc123",
  "requestId": "req_xyz789",
  "service": "api",
  "env": "production"
}
```

| Field | Type | Purpose |
|---|---|---|
| `level` | string | Log severity: debug/info/warn/error |
| `time` | ISO 8601 | When the event occurred (UTC always) |
| `msg` | string | Human-readable description of the event |
| `requestId` | string | Correlates all logs from the same HTTP request |
| `userId` | string | Who the event belongs to (omit for unauthenticated events) |
| `service` | string | Which service or process logged this |
| `env` | string | Which environment |

## Implementation with Pino

```typescript
// lib/logger.ts
import pino from 'pino';

export const logger = pino({
  level: process.env.LOG_LEVEL ?? (process.env.NODE_ENV === 'development' ? 'debug' : 'info'),
  base: {
    service: 'api',
    env: process.env.NODE_ENV,
  },
  timestamp: pino.stdTimeFunctions.isoTime,
  redact: {
    paths: [
      'req.headers.authorization',
      'req.headers.cookie',
      '*.password',
      '*.token',
      '*.apiKey',
      '*.cardNumber',
      '*.cvv',
    ],
    censor: '[REDACTED]',
  },
});

// Request logger middleware
export function requestLogger(req: Request, res: Response, next: NextFunction) {
  const requestId = randomUUID();
  req.requestId = requestId;

  const childLogger = logger.child({ requestId, userId: req.session?.userId });
  req.logger = childLogger;

  const start = Date.now();

  res.on('finish', () => {
    childLogger.info({
      method: req.method,
      path: req.path,  // Not req.url — omits query params which may contain PII
      statusCode: res.statusCode,
      durationMs: Date.now() - start,
    }, 'Request completed');
  });

  next();
}
```

## Log Level Definitions

| Level | Use for |
|---|---|
| `debug` | Detailed diagnostic information; development only |
| `info` | Normal business events: user signed up, payment processed, job completed |
| `warn` | Unexpected but recoverable: retry triggered, deprecated API called, high memory |
| `error` | Failures requiring attention: unhandled exception, payment failed, DB connection dropped |

## What Must Never Appear in Logs

**Never log:**
- Passwords (hashed or plaintext)
- Session tokens, JWTs, auth cookies
- API keys or secrets
- Full credit card numbers, CVV, or PANs
- Raw authentication headers (`Authorization`)
- Request or response bodies (which may contain any of the above)
- User-typed content that may include passwords (form submissions)

The `redact` configuration above handles known paths, but the real protection is not logging raw request bodies at all.

```typescript
// Safe
logger.info({ userId, plan: 'pro' }, 'User upgraded plan');

// Unsafe — never log req.body in production
logger.info({ body: req.body }, 'Received payment request'); // ← Wrong
```

## Security

Logs are a frequent target in breach investigations and breaches themselves. A log file that contains session tokens allows an attacker to impersonate any user whose token was logged. Log only what you would be comfortable with an attacker reading.

## Related Documents

- [Environments logging by environment](../delivery/environments/logging-by-environment.md)
- [Observability error monitoring](error-monitoring.md)
- [Backend error messages](../backend/runtime/error-messages.md)
