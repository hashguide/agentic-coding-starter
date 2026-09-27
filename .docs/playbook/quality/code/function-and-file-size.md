---
kind: playbook
domain: quality
topics: [quality, code, function, and, file, size]
authority: guidance
---

# Function and File Size

Size limits exist because of what size correlates with: a function longer than ~40 lines is almost always doing more than one thing at its level of abstraction. A file with more than 400 lines is almost always a module that has accumulated unrelated responsibilities. The numbers are signals, not rules — but the signal is reliable.

## Function Size Guidance

**Threshold:** A function longer than ~40 lines, or one that does more than one thing at its level of abstraction, should be reconsidered.

This doesn't mean every function must be under 40 lines. A 60-line function that does one thing clearly — perhaps constructing a complex object from many inputs — is fine. A 30-line function that handles validation, database write, email send, and error formatting is too long by any measure.

**The real test:** Can you describe what this function does in one sentence without using "and"? If not, it's doing more than one thing.

**Before:**

```typescript
async function createInvoice(userId: string, data: CreateInvoiceInput) {
  // Validate
  if (!data.clientId) throw new ValidationError('CLIENT_REQUIRED');
  if (!data.lineItems.length) throw new ValidationError('LINE_ITEMS_REQUIRED');
  const totalAmount = data.lineItems.reduce((sum, item) => {
    if (item.quantity <= 0) throw new ValidationError('INVALID_QUANTITY');
    if (item.unitPrice <= 0) throw new ValidationError('INVALID_PRICE');
    return sum + (item.quantity * item.unitPrice);
  }, 0);

  // Database write
  const invoice = await db.invoice.create({
    data: {
      userId,
      clientId: data.clientId,
      total: totalAmount,
      status: 'draft',
      lineItems: { create: data.lineItems },
    },
  });

  // Send notification
  await createNotification({
    userId,
    type: 'invoice_created',
    message: `Invoice #${invoice.id} created`,
    link: `/invoices/${invoice.id}`,
  });

  // Analytics
  analytics.track('invoice_created', { userId, total: totalAmount });

  return invoice;
}
```

**After:**

```typescript
async function createInvoice(userId: string, data: CreateInvoiceInput) {
  const validatedData = validateInvoiceInput(data);
  const invoice = await persistInvoice(userId, validatedData);
  await notifyInvoiceCreated(userId, invoice);
  return invoice;
}

function validateInvoiceInput(data: CreateInvoiceInput): ValidatedInvoiceInput {
  if (!data.clientId) throw new ValidationError('CLIENT_REQUIRED');
  if (!data.lineItems.length) throw new ValidationError('LINE_ITEMS_REQUIRED');
  const total = data.lineItems.reduce((sum, item) => {
    if (item.quantity <= 0) throw new ValidationError('INVALID_QUANTITY');
    if (item.unitPrice <= 0) throw new ValidationError('INVALID_PRICE');
    return sum + (item.quantity * item.unitPrice);
  }, 0);
  return { ...data, total };
}
```

The top-level function now reads like a clear sequence of steps. Each step is testable independently.

## File Size Guidance

**Threshold:** A file approaching 400 lines is a signal that it may have accumulated multiple responsibilities. Files regularly exceeding 400 lines should be audited.

**When large files are acceptable:** A file containing a single complex data structure definition, a long configuration object, or a generated file (e.g., migration files, GraphQL schema) may legitimately be large. The question is not "how long?" but "how many responsibilities?"

**Splitting a large file:** Look for natural seams — groups of functions that all relate to the same sub-concept. Extract those into their own file and re-export from the index.

## Related Documents

- [Module boundaries](module-boundaries.md)
- [Code smells](code-smells.md)
- [Refactoring triggers](refactoring-triggers.md)
