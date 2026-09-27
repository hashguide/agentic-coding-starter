---
kind: playbook
domain: backend
topics: [backend, architecture, structure]
authority: guidance
---

# Backend Folder Structure

Folder structure is not cosmetic — it determines how quickly a developer (or AI agent) can find the right file, understand the codebase's mental model, and add new code without violating existing patterns. Three structures are common in TypeScript backends, each with different trade-offs.

## Option 1: Layered / MVC-Style

Organizes by technical layer: controllers, services, models. All controllers in one directory, all services in another.

```
src/
  controllers/
    invoices.ts
    users.ts
    notifications.ts
  services/
    invoice-service.ts
    user-service.ts
    notification-service.ts
  models/
    invoice.ts
    user.ts
  middleware/
    auth.ts
    rate-limit.ts
  lib/
    db.ts
    email.ts
  routes/
    index.ts
```

**When to use:** Small projects where cross-cutting concerns dominate. Easy to find "all controllers" or "all services." Becomes unwieldy as features grow — adding a feature touches four directories.

**Team size fit:** Solo to small teams on small-to-medium projects.

## Option 2: Feature-Based

Organizes by feature/domain. Everything related to invoices lives in one directory.

```
src/
  features/
    invoices/
      invoices.router.ts
      invoices.service.ts
      invoices.schema.ts
      invoices.test.ts
    notifications/
      notifications.router.ts
      notifications.service.ts
      notifications.test.ts
    auth/
      auth.router.ts
      auth.service.ts
      session.ts
  shared/
    lib/
      db.ts
      email.ts
    middleware/
      require-auth.ts
      rate-limit.ts
  app.ts
```

**When to use:** Products with multiple distinct domains where teams (or individuals) own specific features. Adding a feature means adding one directory. Finding code for a feature means looking in one place.

**Team size fit:** Solo builders who plan to scale, small to medium teams.

## Option 3: Domain-Driven Design

Organizes by bounded context, with explicit domain models, application services, and infrastructure layers per domain.

```
src/
  domains/
    billing/
      application/
        commands/
          create-subscription.ts
          cancel-subscription.ts
        queries/
          get-subscription.ts
      domain/
        subscription.ts
        subscription-events.ts
      infrastructure/
        stripe-billing-repo.ts
        billing-router.ts
    invoicing/
      application/
      domain/
      infrastructure/
  shared/
    infrastructure/
      db.ts
      email.ts
```

**When to use:** Products with complex business logic that benefits from explicit separation of domain logic from infrastructure. Adds significant structure — not justified until the domain is complex enough to warrant it.

**Team size fit:** Medium to large teams, complex domains.

## Recommended Default

**Feature-based structure** is a strong fit when product capabilities are the clearest ownership boundary. It scales from small to medium cleanly, avoids the "everything is scattered across four directories" problem of layered MVC, and doesn't require the upfront investment of DDD.

Add a `shared/` directory for utilities, middleware, and library wrappers that are used across multiple features.

## Related Documents

- [Architecture overview](../../architecture/overview.md)
- [Module boundaries](../../quality/code/module-boundaries.md)
- [Dependency direction](../../quality/code/dependency-direction.md)
- [Modular monolith](../../architecture/system-shape/monolith/modular-monolith.md)
