---
kind: playbook
domain: architecture
topics: [architecture, patterns, structural, decorator]
authority: guidance
---

# Decorator Pattern

The Decorator pattern adds behavior to an object without modifying it or creating a subclass. A decorator wraps the original object, implementing the same interface, and adds functionality before or after delegating to the original. This is distinct from middleware (which operates on a request/response pipeline) — a decorator wraps a specific object, not a pipeline stage.

## Definition

A decorator implements the same interface as the object it wraps, adds its own behavior, and delegates the core operation to the wrapped object.

## Code Example

Adding logging and timing to a service call without modifying the service:

```typescript
interface InvoiceService {
  createInvoice(userId: string, data: CreateInvoiceInput): Promise<Invoice>;
  getInvoice(invoiceId: string, userId: string): Promise<Invoice | null>;
}

class BaseInvoiceService implements InvoiceService {
  async createInvoice(userId: string, data: CreateInvoiceInput): Promise<Invoice> {
    return db.invoice.create({ data: { userId, ...data } });
  }

  async getInvoice(invoiceId: string, userId: string): Promise<Invoice | null> {
    return db.invoice.findFirst({ where: { id: invoiceId, userId, deletedAt: null } });
  }
}

// Logging decorator — adds logging without touching BaseInvoiceService
class LoggingInvoiceService implements InvoiceService {
  constructor(
    private readonly inner: InvoiceService,
    private readonly logger: Logger,
  ) {}

  async createInvoice(userId: string, data: CreateInvoiceInput): Promise<Invoice> {
    const start = Date.now();
    try {
      const result = await this.inner.createInvoice(userId, data);
      this.logger.info({ userId, invoiceId: result.id, ms: Date.now() - start }, 'invoice.created');
      return result;
    } catch (err) {
      this.logger.error({ userId, err }, 'invoice.create.failed');
      throw err;
    }
  }

  async getInvoice(invoiceId: string, userId: string): Promise<Invoice | null> {
    return this.inner.getInvoice(invoiceId, userId);
  }
}

// Compose: base service wrapped with logging
const invoiceService: InvoiceService = new LoggingInvoiceService(
  new BaseInvoiceService(),
  logger,
);
```

Multiple decorators can be stacked: `new CachingInvoiceService(new LoggingInvoiceService(new BaseInvoiceService(), logger), cache)`.

## Distinction from Middleware

Middleware operates on a request/response pipeline — it intercepts every request before it reaches a handler. A decorator wraps a specific object instance. Use middleware for cross-cutting concerns that apply to all routes (authentication, rate limiting); use a decorator for concerns that apply to a specific service or object (logging a specific service's calls, caching a specific data source).

## When to Use

- You want to add logging, caching, timing, or retry logic to an existing service without modifying it
- You want to be able to enable/disable the added behavior by composing or removing the decorator
- You need to test the base service without the added behavior, or test the decorator independently

## When to Prefer a Direct Approach

If the added behavior is simple enough — a try/catch wrapper around one function — inline it rather than creating a full decorator class. The decorator is worth the ceremony when the behavior applies to multiple methods of the same interface.

## Related Documents

- [Middleware pattern](../behavioral/middleware.md)
- [Proxy pattern](proxy.md)
- [Patterns overview](../overview.md)
