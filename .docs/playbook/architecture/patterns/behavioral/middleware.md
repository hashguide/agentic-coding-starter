---
kind: playbook
domain: architecture
topics: [architecture, patterns, behavioral, middleware]
authority: guidance
---

# Middleware Pattern

The Middleware pattern (a specific form of the Chain of Responsibility pattern) arranges a series of handlers in a pipeline. Each handler receives a request, optionally performs an action, and either passes the request to the next handler or terminates the chain. In web applications, middleware is how cross-cutting concerns — authentication, logging, rate limiting, validation — are applied without embedding them in every route handler.

## Definition

Middleware is a function (or class) that sits in a request pipeline. It receives the request and a `next` function. It can inspect and modify the request, call `next()` to pass control to the next middleware, or respond directly to terminate the chain.

## Code Example

A middleware chain: auth → rate limit → validation → handler

```typescript
import express, { Request, Response, NextFunction } from 'express';

// 1. Auth middleware — verifies session, attaches user to request
async function requireAuth(req: Request, res: Response, next: NextFunction): Promise<void> {
  const sessionId = req.cookies['session'];
  if (!sessionId) {
    res.status(401).json({ code: 'AUTH_REQUIRED' });
    return;
  }

  const session = await db.session.findUnique({ where: { id: sessionId } });
  if (!session || session.expiresAt < new Date()) {
    res.status(401).json({ code: 'AUTH_REQUIRED' });
    return;
  }

  req.userId = session.userId;
  next(); // passes to next middleware
}

// 2. Rate limit middleware — checks request count per user
async function rateLimit(req: Request, res: Response, next: NextFunction): Promise<void> {
  const key = `rate:${req.userId}`;
  const count = await redis.incr(key);
  if (count === 1) await redis.expire(key, 60); // 1-minute window

  if (count > 100) {
    // Never reveal the limit parameters in the response
    res.status(429).json({ code: 'RATE_LIMITED' });
    return;
  }

  next();
}

// 3. Validation middleware — checks request body schema
function validateBody<T>(schema: z.ZodSchema<T>) {
  return (req: Request, res: Response, next: NextFunction): void => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      res.status(400).json({ code: 'VALIDATION_ERROR', fields: result.error.flatten().fieldErrors });
      return;
    }
    req.body = result.data;
    next();
  };
}

// 4. Route handler — only reached after all middleware passes
const createInvoiceSchema = z.object({
  clientId: z.string().uuid(),
  lineItems: z.array(z.object({
    description: z.string().min(1),
    quantity: z.number().positive(),
    unitPrice: z.number().positive(),
  })).min(1),
});

app.post(
  '/api/invoices',
  requireAuth,
  rateLimit,
  validateBody(createInvoiceSchema),
  async (req, res) => {
    const invoice = await invoiceRepo.create({ userId: req.userId!, ...req.body });
    res.json(invoice);
  },
);
```

## Cross-Reference to API Design

The middleware chain described here is the implementation mechanism for the security and validation requirements in [architecture/backend/api-design.md](../../../backend/architecture/api-design.md). Auth middleware is the enforcement point for the server-side role verification rule in [auth/rbac.md](../../../access/authentication/rbac.md).

## When to Use Middleware vs. In-Handler Logic

| Concern | Middleware | In-handler |
|---|---|---|
| Auth verification | Yes — applies to many routes | No |
| Rate limiting | Yes — cross-cutting | No |
| Request logging | Yes — cross-cutting | No |
| Business validation | Sometimes — shared schemas | Often — business-specific |
| Response formatting | Rarely — use helpers | Yes |

## Related Documents

- [Decorator pattern](../structural/decorator.md)
- [API design](../../../backend/architecture/api-design.md)
- [Auth RBAC](../../../access/authentication/rbac.md)
