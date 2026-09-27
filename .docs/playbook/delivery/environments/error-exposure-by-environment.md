---
kind: playbook
domain: delivery
topics: [delivery, environments, error, exposure, by, environment]
authority: guidance
---

# Error Exposure by Environment

What an error response reveals must vary by environment. Development needs full detail to debug quickly. Production must never expose stack traces, internal field names, or any information that helps an attacker understand the system.

## Error Detail by Environment

| Environment | Error detail level | What the response includes |
|---|---|---|
| **Local** | Full | Stack trace, internal error message, field names, query text — anything that helps debugging |
| **Staging** | Production-equivalent | Same restraint as production by default. Optional: add an `X-Internal-Error-Id` header that developers can correlate to full log output |
| **Production** | Safe, ambiguous | User-facing message + error code + reference ID. No stack traces, no internal field names, no implementation details |

Staging deliberately uses production-equivalent error responses. Staging that shows verbose errors creates a false sense of security — if an error path works correctly in staging (no sensitive data exposed), it works correctly in production.

## Implementation

```typescript
// middleware/errorHandler.ts
import { Request, Response, NextFunction } from 'express';
import { logger } from '../lib/logger';
import { randomBytes } from 'crypto';

export function errorHandler(err: Error, req: Request, res: Response, next: NextFunction) {
  const referenceId = randomBytes(4).toString('hex').toUpperCase();

  // Always log the full error with context
  logger.error({
    err,
    referenceId,
    path: req.path,
    method: req.method,
    userId: req.session?.userId,
  }, 'Unhandled error');

  if (process.env.NODE_ENV === 'development') {
    // In development: full detail is useful
    return res.status(500).json({
      error: 'INTERNAL_SERVER_ERROR',
      message: err.message,
      stack: err.stack,
      referenceId,
    });
  }

  // In staging and production: safe, ambiguous response
  return res.status(500).json({
    error: 'INTERNAL_SERVER_ERROR',
    message: 'Something went wrong. Please try again or contact support.',
    referenceId,
  });
}
```

The reference ID allows the user or support team to correlate a specific incident to the full log entry without exposing the details in the response.

## Why This Must Be Enforced in Code

Environment-specific error exposure enforced by convention fails:
- Under deadline pressure, verbose error logging gets added "temporarily" and never removed
- A misconfigured environment variable can cause the wrong environment to behave like development
- A new developer may not know the convention exists

The error handler itself checks `process.env.NODE_ENV`. There is no way to accidentally expose stack traces in production without also having incorrectly set `NODE_ENV=development` in production — which would fail the startup env validation.

## Security

Environment-specific error exposure must be enforced in the error handler itself — convention alone breaks under deadline pressure. Stack traces and internal field names help attackers map the system. A reference ID gives support enough context without exposing implementation details.

## Related Documents

- [Backend error messages](../../backend/runtime/error-messages.md)
- [Environments logging by environment](logging-by-environment.md)
- [Environments production](production.md)
