---
kind: playbook
domain: quality
topics: [quality, code, module, boundaries]
authority: guidance
---

# Module Boundaries

A module owns a domain. It exposes a public interface to other modules and keeps its implementation private. Other modules interact with the public interface — they never reach into the internals. Boundary violations are how "one small change" becomes a cascade of unintended consequences across the codebase.

## Public Interface vs. Internal Implementation

**Public interface:** The exports that are intentionally available to other modules. If a module exports ten functions, the public interface is the two or three that other modules should actually use.

**Internal implementation:** The helpers, utilities, and sub-functions that the module uses internally. These should not be exported — if they are, other modules will eventually use them, creating hidden coupling.

In TypeScript:

```typescript
// src/notifications/index.ts — public interface
export { createNotification } from './create';
export { listNotifications } from './list';
export { markRead, markAllRead } from './read';
export type { Notification, NotificationType } from './types';

// Not exported (internal):
// buildNotificationPayload() — implementation detail
// validateNotificationType() — internal validation
```

Other parts of the codebase import only from `src/notifications`, never from `src/notifications/create` directly. This ensures the notification module can reorganize its internals without affecting callers.

## Boundary Violation Example

**Violation:**

```typescript
// src/billing/checkout.ts — WRONG: reaches into notifications internals
import { buildNotificationPayload } from '../notifications/create';

// ... uses the internal helper directly
```

This creates hidden coupling. If `buildNotificationPayload` is renamed, moved, or its signature changes, `billing/checkout.ts` breaks — even though it's not part of the notifications module.

**Fix:**

```typescript
// src/notifications/index.ts — expose what billing actually needs
export { createNotification } from './create';

// src/billing/checkout.ts — CORRECT: uses the public interface
import { createNotification } from '../notifications';

await createNotification({
  userId,
  type: 'payment_succeeded',
  message: 'Your payment was processed.',
  link: `/invoices/${invoiceId}`,
});
```

## When a Boundary Violation Is Tempting

Boundary violations usually happen under one of three conditions:

1. **The public interface doesn't expose what you need.** Solution: add the function to the public interface, or (if it's genuinely internal) refactor the internals so the caller's use case is handled internally.
2. **The module boundary is wrong.** Sometimes two things that are in separate modules belong in the same module. Consolidate rather than crossing the boundary.
3. **Deadline pressure.** The fastest path is to import the internal. The correct path is to fix the interface. Fix the interface.

## Related Documents

- [Dependency direction](dependency-direction.md)
- [Function and file size](function-and-file-size.md)
- [Architecture overview](../../architecture/overview.md)
