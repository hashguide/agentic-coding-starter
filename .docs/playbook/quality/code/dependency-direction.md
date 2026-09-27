---
kind: playbook
domain: quality
topics: [quality, code, dependency, direction]
authority: guidance
---

# Dependency Direction

Dependencies flow in one direction: from higher-level modules toward lower-level ones. Features depend on shared services; shared services depend on the data layer; nothing lower-level depends on something higher-level. Circular dependencies — where A depends on B and B depends on A — and inverted dependencies — where a utility imports a feature — create import cycles, initialization order bugs, and testing nightmares.

## Layered Dependency Diagram

```
┌─────────────────────────────────────┐
│  Features / Route Handlers          │  ← depends on ↓
├─────────────────────────────────────┤
│  Services / Business Logic          │  ← depends on ↓
├─────────────────────────────────────┤
│  Shared Libraries (lib/)            │  ← depends on ↓
├─────────────────────────────────────┤
│  Data Layer (ORM / DB clients)      │  ← depends on ↓
├─────────────────────────────────────┤
│  External Services / Infrastructure │
└─────────────────────────────────────┘
```

**Rule:** Dependencies only point downward. A lower layer never imports from a higher layer.

**Feature → Service → Data is correct.**
**Data → Service or Data → Feature is wrong.**

In practice for a TypeScript app:

```
src/routes/invoices.ts         → src/services/invoice-service.ts
src/services/invoice-service.ts → src/lib/email.ts, src/db/invoice-repo.ts
src/db/invoice-repo.ts          → src/lib/db.ts (Prisma client)
src/lib/db.ts                   → @prisma/client (external)
```

`src/lib/db.ts` must never import from `src/services/` or `src/routes/`. If it appears to need to, the dependency direction is inverted and the design needs to change.

## How Circular Dependencies Get Introduced

The most common cause: two services that legitimately need each other's functionality.

```
// invoice-service.ts needs to send email via email-service.ts
// email-service.ts needs to look up user prefs via user-service.ts
// user-service.ts needs invoice data from invoice-service.ts
// → circular
```

The fix is never "live with the cycle" — it's to identify the shared dependency and extract it downward:

```
// Extract the shared lookup into a lower-level utility
// email-service.ts and invoice-service.ts both import the utility
// Neither imports the other
```

Or: use dependency injection / an event system to break the direct dependency. Invoice service emits `invoice.created` event; email service subscribes. Neither knows about the other.

## UI Equivalent

In a React frontend:

```
pages/           → features/
features/        → components/shared/
components/      → lib/ (hooks, utilities)
lib/             → (nothing in this project)
```

A shared `Button` component must never import from a feature. If a shared component needs feature-specific behavior, that behavior should be passed in as a prop — the feature reaches into the component, not the other way around.

## Detecting Violations

```bash
# Use madge (Node.js dependency graph tool) to find circular dependencies
npx madge --circular src/
```

## Related Documents

- [Module boundaries](module-boundaries.md)
- [Code quality overview](overview.md)
- [Architecture backend structure](../../backend/architecture/structure.md)
