---
kind: playbook
domain: architecture
topics: [architecture, patterns, creational, builder]
authority: guidance
---

# Builder Pattern

The Builder pattern constructs complex objects step by step. It separates the construction of an object from its representation, allowing the same construction process to produce different results based on the steps called. It's most useful when an object requires many configuration choices, some of which are optional, and when the object is invalid until construction is complete.

## Definition

A builder accumulates configuration through a series of method calls, then produces the final object when a terminal method (typically `build()`) is called. Each method returns the builder itself, enabling method chaining.

## Code Example

Constructing a complex database query where filters, pagination, and sorting are all optional:

```typescript
class InvoiceQueryBuilder {
  private filters: Prisma.InvoiceWhereInput = {};
  private orderByClause: Prisma.InvoiceOrderByWithRelationInput = { createdAt: 'desc' };
  private paginationParams = { skip: 0, take: 20 };
  private includeClause: Prisma.InvoiceInclude = {};

  forUser(userId: string): this {
    this.filters.userId = userId;
    return this;
  }

  withStatus(status: InvoiceStatus): this {
    this.filters.status = status;
    return this;
  }

  withClient(clientId: string): this {
    this.filters.clientId = clientId;
    return this;
  }

  sortBy(field: 'createdAt' | 'dueDate' | 'total', direction: 'asc' | 'desc'): this {
    this.orderByClause = { [field]: direction };
    return this;
  }

  paginate(page: number, pageSize = 20): this {
    this.paginationParams = { skip: (page - 1) * pageSize, take: pageSize };
    return this;
  }

  withLineItems(): this {
    this.includeClause.lineItems = true;
    return this;
  }

  async build(): Promise<Invoice[]> {
    return db.invoice.findMany({
      where: { ...this.filters, deletedAt: null },
      orderBy: this.orderByClause,
      ...this.paginationParams,
      include: this.includeClause,
    });
  }
}

// Usage — only specify what you need
const invoices = await new InvoiceQueryBuilder()
  .forUser(userId)
  .withStatus('pending')
  .sortBy('dueDate', 'asc')
  .paginate(2)
  .withLineItems()
  .build();
```

## When a Builder Is Justified

- The object requires five or more configuration options, many of which are optional
- The options interact with each other and the builder can validate their combination
- You want a fluent, readable API for complex construction
- Construction is multi-step and intermediate states should not be exposed

## When a Plain Object Literal Suffices

For most configuration objects, a TypeScript interface with optional properties is simpler and more readable than a builder:

```typescript
// Usually better for simple cases
const invoices = await queryInvoices({
  userId,
  status: 'pending',
  sortBy: 'dueDate',
  page: 2,
  includeLineItems: true,
});
```

The builder pattern adds value when the combination of options is complex enough that the method chain reads more clearly than an object literal, or when the builder can enforce ordering constraints (e.g., `paginate()` must come after `forUser()`).

## Related Documents

- [Patterns overview](../overview.md)
- [Factory pattern](factory.md)
