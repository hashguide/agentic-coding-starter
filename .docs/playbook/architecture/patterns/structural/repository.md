---
kind: playbook
domain: architecture
topics: [architecture, patterns, structural, repository]
authority: guidance
---

# Repository Pattern

The Repository pattern abstracts data access behind an interface, centralizing all queries for a given entity in one place. Instead of scattering ORM calls across services and route handlers, every database operation for a given entity goes through its repository. The rest of the application sees a clean, queryable interface and never touches the ORM directly.

## Definition

A repository is a class that encapsulates all data access logic for a single entity — queries, creates, updates, and deletes — and exposes them through a typed, business-domain-oriented interface.

## Code Example

A repository class wrapping ORM calls for the Invoice entity:

```typescript
import { Prisma } from '@prisma/client';
import { db } from '../lib/db';

export class InvoiceRepository {
  async findById(id: string, userId: string): Promise<Invoice | null> {
    return db.invoice.findFirst({
      where: { id, userId, deletedAt: null },
      include: { lineItems: true },
    });
  }

  async findAllForUser(
    userId: string,
    options: { status?: InvoiceStatus; page?: number; pageSize?: number } = {},
  ): Promise<Invoice[]> {
    const { status, page = 1, pageSize = 20 } = options;
    return db.invoice.findMany({
      where: {
        userId,
        deletedAt: null,
        ...(status ? { status } : {}),
      },
      include: { lineItems: true },
      orderBy: { createdAt: 'desc' },
      skip: (page - 1) * pageSize,
      take: pageSize,
    });
  }

  async create(data: {
    userId: string;
    clientId: string;
    lineItems: Array<{ description: string; quantity: number; unitPrice: number }>;
  }): Promise<Invoice> {
    const total = data.lineItems.reduce(
      (sum, item) => sum + item.quantity * item.unitPrice,
      0,
    );

    return db.invoice.create({
      data: {
        userId: data.userId,
        clientId: data.clientId,
        total,
        status: 'draft',
        lineItems: { create: data.lineItems },
      },
      include: { lineItems: true },
    });
  }

  async softDelete(id: string, userId: string): Promise<void> {
    await db.invoice.updateMany({
      where: { id, userId, deletedAt: null },
      data: { deletedAt: new Date() },
    });
  }
}
```

Services and route handlers use the repository:

```typescript
// In a service or route handler:
const repo = new InvoiceRepository();
const invoice = await repo.findById(invoiceId, userId);
if (!invoice) throw new NotFoundError('INVOICE_NOT_FOUND');
```

## Benefits

**Centralized query logic:** All queries for a given entity are in one file. When the schema changes, you update the repository, not every file that happened to query invoices.

**Easier testing:** In unit tests, replace the repository with a mock. The service under test doesn't need a real database connection.

**Swappable data layer:** If you change ORMs, you rewrite the repository implementations, not the services that use them.

## When It's Unnecessary Overhead

For a small project with one or two entities and simple queries, a repository layer adds files and indirection without meaningful benefit. The pattern earns its keep when:
- Multiple services query the same entity
- Query logic is complex enough that duplication would be a real risk
- You want to unit test services without database connections

For a small solo project, writing ORM calls directly in service functions is fine. Add the repository layer when duplication or testability becomes a real concern.

## Related Documents

- [Database ORM](../../../data/orm.md)
- [Database schema design](../../../data/schema-design.md)
- [Testing integration](../../../quality/testing/integration.md)
