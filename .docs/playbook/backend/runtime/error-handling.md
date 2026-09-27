---
kind: playbook
domain: backend
topics: [backend, runtime, error, handling]
authority: guidance
---

# Error Handling

A production backend has two error handling requirements: internal errors must be logged with full detail for debugging, and external responses must be sanitized so attackers learn nothing from error messages. A global error handler at the application boundary is where these two requirements meet.

## Error Class Hierarchy

Define a typed error hierarchy so catch blocks can make meaningful decisions based on error type.

```typescript
// Base error class — all application errors extend this
export class AppError extends Error {
  constructor(
    public readonly code: string,
    public readonly statusCode: number,
    message: string,
    public readonly details?: unknown,
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

export class ValidationError extends AppError {
  constructor(code: string, fields?: Record<string, string[]>) {
    super(code, 400, 'Validation failed', fields);
  }
}

export class NotFoundError extends AppError {
  constructor(code: string) {
    super(code, 404, 'Resource not found');
  }
}

export class AuthError extends AppError {
  constructor() {
    // Always AUTH_FAILED — never reveal why (email not found vs wrong password)
    super('AUTH_FAILED', 401, 'Authentication failed');
  }
}

export class ForbiddenError extends AppError {
  constructor(code = 'FORBIDDEN') {
    super(code, 403, 'Access denied');
  }
}

export class RateLimitError extends AppError {
  constructor() {
    // Never reveal limit parameters, window, or remaining attempts
    super('RATE_LIMITED', 429, 'Too many requests');
  }
}
```

## Global Exception Handler

The global error handler is registered as the last middleware in the Express (or equivalent) chain. It is the final safety net — any error that reaches here is sanitized before the response is sent.

```typescript
import { Request, Response, NextFunction } from 'express';
import { AppError } from './errors';
import { logger } from './lib/logger';

export function globalErrorHandler(
  err: unknown,
  req: Request,
  res: Response,
  _next: NextFunction,
): void {
  // Generate a reference ID for correlating logs to user reports
  const errorId = crypto.randomUUID();

  if (err instanceof AppError) {
    // Known error: log at warn level with structured context
    logger.warn({
      errorId,
      code: err.code,
      statusCode: err.statusCode,
      path: req.path,
      userId: req.userId, // Attached by auth middleware
    }, err.message);

    // Send structured error response — no stack trace, no internals
    res.status(err.statusCode).json({
      code: err.code,
      ...(err.details ? { fields: err.details } : {}),
      errorId, // Reference ID the user can report
    });
    return;
  }

  // Unknown error: log at error level with full stack trace (server-side only)
  logger.error({
    errorId,
    err, // Includes stack trace — stays in logs, never sent to client
    path: req.path,
    userId: req.userId,
  }, 'Unhandled error');

  // Generic response — reveals nothing about the internal error
  res.status(500).json({
    code: 'INTERNAL_ERROR',
    errorId, // Reference ID only — no stack trace, no internals
  });
}

// Register as the last middleware
app.use(globalErrorHandler);
```

## Handling Async Errors in Route Handlers

Express does not automatically pass async errors to the error handler. Wrap async handlers:

```typescript
// Utility to wrap async route handlers
function asyncHandler(fn: (req: Request, res: Response, next: NextFunction) => Promise<void>) {
  return (req: Request, res: Response, next: NextFunction) => {
    fn(req, res, next).catch(next);
  };
}

app.get('/api/invoices/:id', requireAuth, asyncHandler(async (req, res) => {
  const invoice = await invoiceRepo.findById(req.params.id, req.userId!);
  if (!invoice) throw new NotFoundError('INVOICE_NOT_FOUND');
  res.json(invoice);
}));
```

## Security

The global error handler sanitizes all errors before sending. Stack traces, internal field names, database error messages, and ORM internals never reach the client response. This is not just good practice — it prevents information disclosure that aids attackers.

Specifically:
- Database error messages often contain schema information (table names, column names)
- Stack traces reveal file paths, function names, and application structure
- ORM errors may include the raw query that failed

None of this belongs in an API response. The `errorId` reference ID is the bridge: the user can report it, and you can look up the full details in your logs.

## Related Documents

- [Error messages](error-messages.md)
- [Backend logging](logging.md)
- [Security checklist](../../security/checklist.md)
