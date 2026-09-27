---
kind: playbook
domain: architecture
topics: [architecture, system, shape, monolith, modular]
authority: guidance
---

# Modular Monolith

A modular monolith is a single deployable unit whose internal code is organized into modules with explicit, enforced boundaries. Modules communicate only through defined public interfaces — not by importing each other's internals. The code structure inside the monolith mirrors what the code would look like if it were separated into services, without the operational cost of actually separating them.

## Why Structure Within a Monolith Matters

An unstructured monolith — where any file can import any other file — accumulates coupling. After two years, changing one feature requires understanding the ripple effects through the entire codebase. A modular monolith prevents this by treating module boundaries as a first-class constraint.

## Folder Structure Enforcing Module Boundaries

```
src/
  modules/
    billing/
      index.ts          ← public interface: exports BillingService only
      billing.service.ts
      billing.repo.ts
      stripe.client.ts  ← internal, not exported
    notifications/
      index.ts          ← public interface: exports createNotification, listNotifications
      notification.service.ts
      notification.repo.ts
      push.client.ts    ← internal
    invoicing/
      index.ts          ← public interface: exports InvoiceService
      invoice.service.ts
      invoice.repo.ts
  shared/
    lib/
      db.ts             ← shared infrastructure (Prisma client)
      email.ts          ← shared infrastructure (email sender)
      logger.ts
    middleware/
      require-auth.ts
      rate-limit.ts
  app.ts
  router.ts
```

**Rule:** Modules import only from other modules' `index.ts` files (public interface). Never import directly from internal files of another module.

```typescript
// CORRECT: billing service imports through invoicing's public interface
import { InvoiceService } from '../invoicing'; // imports from index.ts

// WRONG: billing service reaches into invoicing's internals
import { calculateInvoiceTotal } from '../invoicing/invoice.service'; // ← boundary violation
```

## Using Internal Package Boundaries

For stronger enforcement, use a monorepo with workspace packages. Each module is a workspace package; imports from internal paths in other packages fail at build time.

```
packages/
  billing/
    src/
      index.ts    ← only this is importable by other packages
    package.json  ← name: "@app/billing"
  notifications/
    src/
      index.ts
    package.json  ← name: "@app/notifications"
  invoicing/
    src/
      index.ts
    package.json  ← name: "@app/invoicing"
  web/
    package.json  ← depends on "@app/billing", "@app/notifications", "@app/invoicing"
```

With a monorepo, attempting to import `@app/billing/src/stripe.client` fails — only `@app/billing` is the public import path.

Tools: Turborepo, Nx, pnpm workspaces.

## Cross-Reference to Dependency Direction

The module boundary rules here enforce the dependency direction rules from [code-quality/dependency-direction.md](../../../quality/code/dependency-direction.md). A module in `shared/` must never import from `modules/` — it's a lower layer and cannot depend on a higher layer.

## Related Documents

- [Architecture overview](../../overview.md)
- [Monolith overview](overview.md)
- [Module boundaries](../../../quality/code/module-boundaries.md)
- [Dependency direction](../../../quality/code/dependency-direction.md)
