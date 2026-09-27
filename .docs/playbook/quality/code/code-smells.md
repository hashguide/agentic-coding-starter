---
kind: playbook
domain: quality
topics: [quality, code, smells]
authority: guidance
---

# Code Smells

Code smells are structural signals that the code is likely to be hard to maintain, extend, or understand — even when it currently works. They don't mean the code is wrong; they mean the design has a problem that will compound over time. Naming them precisely is the first step to fixing them.

## God Object

**Description:** A class or module that knows too much or does too much. It has become the catch-all for logic that doesn't fit elsewhere. Every other module depends on it.

**Example:**
```typescript
// UserManager handles auth, profile, billing, notifications, and email
class UserManager {
  login() {}
  logout() {}
  updateProfile() {}
  cancelSubscription() {}
  sendWelcomeEmail() {}
  createNotification() {}
  exportData() {}
}
```

**Fix direction:** Decompose by responsibility. Each responsibility becomes its own module or service with a focused interface.

## Shotgun Surgery

**Description:** A single change in requirements requires small edits in many unrelated files. You can't change one thing without touching five.

**Example:** Adding a new notification type requires adding a case to: a database enum migration, a notification handler, a display name mapping in the UI, an email template switch, and a test fixture factory — each in a separate, unrelated file.

**Fix direction:** Centralize the variation point. A single `NOTIFICATION_TYPES` registry that drives all downstream behavior.

## Feature Envy

**Description:** A function that uses the data or methods of another module more than its own. It belongs in the other module.

**Example:**
```typescript
// In invoice-service.ts — uses client data extensively
function formatInvoiceGreeting(invoice: Invoice, client: Client) {
  const fullName = `${client.firstName} ${client.lastName}`;
  const address = formatAddress(client.street, client.city, client.country);
  return `Dear ${fullName},\n${address}`;
}
```

This function is clearly more interested in `Client` than in `Invoice`. It belongs in a client utility module.

## Deep Nesting

**Description:** Logic buried three or more levels of indentation deep — nested conditionals inside loops inside try/catch blocks. Deeply nested code is hard to read, hard to test, and error-prone.

**Fix direction:** Early returns, guard clauses, and extracted functions eliminate nesting.

```typescript
// Deep nesting — hard to follow
function processPayment(user: User, invoice: Invoice) {
  if (user.isActive) {
    if (invoice.status === 'pending') {
      if (invoice.total > 0) {
        // 3 levels in before we get to the actual work
        return chargeCard(user, invoice);
      }
    }
  }
}

// Guard clauses — same logic, readable
function processPayment(user: User, invoice: Invoice) {
  if (!user.isActive) return;
  if (invoice.status !== 'pending') return;
  if (invoice.total <= 0) return;
  return chargeCard(user, invoice);
}
```

## Primitive Obsession

**Description:** Using primitive types (strings, numbers, booleans) where a named type or small value object would clarify meaning.

**Example:**
```typescript
// What is this string? A user ID? An invoice ID? A Stripe customer ID?
function chargeUser(id: string, amount: number, currency: string) {}

// Clearer with typed values
function chargeUser(userId: UserId, amount: Money, currency: Currency) {}
```

In TypeScript, use branded types or simple wrapper types to give semantics to primitives that cross module boundaries.

## Long Parameter Lists

**Description:** A function with four or more parameters is harder to call correctly and harder to extend without breaking callers.

**Fix direction:** Group related parameters into an options object.

```typescript
// Hard to call correctly — what order are these in?
function sendEmail(to: string, subject: string, body: string, from: string, replyTo: string) {}

// Named, extensible
function sendEmail(options: {
  to: string;
  subject: string;
  body: string;
  from?: string;
  replyTo?: string;
}) {}
```

## Related Documents

- [Function and file size](function-and-file-size.md)
- [Refactoring triggers](refactoring-triggers.md)
- [Module boundaries](module-boundaries.md)
