---
kind: playbook
domain: quality
topics: [quality, code, naming, conventions]
authority: guidance
---

# Naming Conventions

Inconsistent naming is the lowest-impact individual decision and the highest-impact accumulated problem in a codebase. A developer reading a file should never need to wonder which convention applies here. Naming inconsistency compounds: once a new pattern appears without correction, it becomes an implicit alternative that spreads.

## Convention Table

| Category | Convention | Example |
|---|---|---|
| Files (non-component) | `kebab-case.ts` | `user-service.ts`, `auth-middleware.ts` |
| React component files | `PascalCase.tsx` | `NotificationBell.tsx`, `InvoiceList.tsx` |
| React components | `PascalCase` | `function NotificationBell()` |
| Functions | `camelCase` | `sendPasswordResetEmail()`, `getInvoiceById()` |
| Variables | `camelCase` | `const invoiceTotal`, `let hasUnreadNotifications` |
| Constants | `SCREAMING_SNAKE_CASE` | `const MAX_RETRY_ATTEMPTS = 3` |
| TypeScript types | `PascalCase` | `type InvoiceStatus`, `interface UserProfile` |
| TypeScript enums | `PascalCase` (enum), `PascalCase` (values) | `enum SubscriptionStatus { Active, Canceled }` |
| Database table names | `snake_case`, plural | `users`, `invoice_line_items` |
| Database column names | `snake_case` | `created_at`, `deleted_at`, `user_id` |
| CSS class names | `kebab-case` | `.notification-bell`, `.invoice-row` |
| Environment variables | `SCREAMING_SNAKE_CASE` | `DATABASE_URL`, `STRIPE_SECRET_KEY` |
| URL paths | `kebab-case` | `/api/invoice-line-items`, `/settings/billing` |

These are the TypeScript/React/Postgres defaults used throughout this playbook. Adapt for other stacks, but document the adaptations in your project's `CLAUDE.md`.

## Naming Correctness — Beyond Convention

Convention governs format. Naming correctness governs meaning. A correctly-formatted name that says the wrong thing is worse than a malformed name — the reader trusts the wrong information.

**Common naming mistakes:**

| Bad name | Why it's wrong | Better name |
|---|---|---|
| `data` | Generic; says nothing about what's in it | `invoices`, `userProfile`, `webhookPayload` |
| `temp` | Suggests something will be renamed later; it won't | Use the actual name now |
| `handle` | Prefix adds nothing | `processWebhookEvent()`, `markNotificationRead()` |
| `flag` | Binary names should state what the flag means | `isEmailVerified`, `hasActiveSubscription` |
| `info` | Generic suffix | `billingDetails`, `sessionMetadata` |

**Boolean naming rule:** Boolean variables and functions should read as a true/false question: `isLoading`, `hasErrors`, `canEdit`, `shouldRedirect`. Not `loading`, `errors`, `edit`.

## Naming Inconsistency Compounds

When a naming inconsistency appears — even a small one — it must be corrected immediately, not scheduled for "later cleanup." Inconsistencies left in place become the pattern for the next person who writes code in that file. Three files using `user_id` and one using `userId` becomes five files using `userId` once an agent reads only the inconsistent one.

The Boy Scout Rule applies here: leave the naming in a file better than you found it. Rename a variable or function while you're already modifying the file — it's a one-line change in the PR you're already writing.

## Related Documents

- [Consistency enforcement](consistency-enforcement.md)
- [Code quality overview](overview.md)
- [Codebase onboarding](codebase-onboarding.md)
