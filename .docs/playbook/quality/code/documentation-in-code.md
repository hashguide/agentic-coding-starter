---
kind: playbook
domain: quality
topics: [quality, code, documentation, in]
authority: guidance
---

# Documentation in Code

Good code is mostly self-documenting. Well-named functions, clearly-typed parameters, and short focused implementations communicate what the code does without any comment needed. Comments are for the cases where the code cannot communicate the why on its own — a constraint that isn't visible in the implementation, a workaround for an external bug, a non-obvious invariant.

## Comment Why, Not What

The code shows what it does. Comments explain why it does it that way.

**Bad (restates the code):**
```typescript
// Increment the count
count++;

// Check if the user is active
if (user.isActive) {
```

**Good (explains non-obvious reasoning):**
```typescript
// Stripe's idempotency key expires after 24 hours, so we include the date
// to ensure a new key is generated on retry after expiry.
const idempotencyKey = `invoice-${invoice.id}-${todayDate}`;

// RLS policies filter by tenant_id automatically, but the admin role
// bypasses RLS — this explicit check is required for admin-context queries.
if (context.isAdmin) {
  where.tenantId = context.tenantId;
}
```

## When Comments Signal a Code Problem

A comment that explains confusing code is a signal that the code itself should be clarified instead. If you find yourself writing a multi-line comment explaining how a function works, consider:

1. Can the function be renamed to communicate its purpose?
2. Can a confusing conditional be extracted into a named helper function?
3. Can a complex expression be split into named variables?

```typescript
// Bad: comment explaining unclear code
// If status is 1 or 4 but not 2, and the user has a premium flag
if ((status === 1 || status === 4) && status !== 2 && user.premium) {

// Good: self-documenting code eliminates the need for the comment
const isEligibleForExport = (status === InvoiceStatus.Draft || status === InvoiceStatus.Sent)
  && user.hasPremiumPlan;
if (isEligibleForExport) {
```

## Docstring Format for Public Functions

Public functions in shared modules benefit from a brief docstring. One or two lines is enough for most functions. Use TSDoc format for TypeScript:

```typescript
/**
 * Marks all unread notifications as read for the given user.
 * Does not delete them — use `deleteNotification` for deletion.
 */
export async function markAllNotificationsRead(userId: string): Promise<void> {
  await db.notification.updateMany({
    where: { userId, read: false },
    data: { read: true },
  });
}
```

Docstrings are most valuable on:
- Public module exports that other modules call
- Functions with non-obvious parameter constraints ("must be a valid UUID," "amount in cents not dollars")
- Functions where a near-identical alternative exists and the distinction matters

Do not write docstrings for internal implementation functions — those change too frequently and docstrings become outdated misleading comments.

## Related Documents

- [Naming conventions](naming-conventions.md)
- [Codebase onboarding](codebase-onboarding.md)
- [Code smells](code-smells.md)
