---
kind: playbook
domain: frontend
topics: [frontend, structure]
authority: guidance
---

# Frontend Folder Structure

Frontend folder structure determines how quickly a developer can find a component, understand where state lives, and add new features without breaking existing ones. Three structures cover most TypeScript/React applications.

## Option 1: Feature-Based

Groups all files related to a feature — components, hooks, services, types, tests — in one directory.

```
src/
  features/
    invoices/
      InvoiceList.tsx
      InvoiceDetail.tsx
      InvoiceForm.tsx
      useInvoices.ts
      invoices.api.ts
      invoices.types.ts
      InvoiceList.test.tsx
    notifications/
      NotificationBell.tsx
      NotificationPanel.tsx
      useNotifications.ts
      notifications.api.ts
    auth/
      LoginPage.tsx
      SignupPage.tsx
      useSession.ts
  shared/
    components/
      Button.tsx
      Input.tsx
      Modal.tsx
    hooks/
      useDebounce.ts
      useLocalStorage.ts
    lib/
      api-client.ts
      date-utils.ts
  app/
    layout.tsx
    page.tsx
    router.tsx
```

**When to use:** Products with multiple distinct features where feature ownership matters. Adding a feature means adding one directory. Scales cleanly from small to large.

**Team size fit:** Solo builders who plan to scale, small to medium teams.

## Option 2: Atomic Design

Organizes components by abstraction level: atoms (basic elements), molecules (composed atoms), organisms (complex components), templates (page layouts), pages.

```
src/
  components/
    atoms/
      Button.tsx
      Input.tsx
      Badge.tsx
    molecules/
      FormField.tsx
      NotificationItem.tsx
      InvoiceRow.tsx
    organisms/
      NotificationPanel.tsx
      InvoiceList.tsx
      InvoiceForm.tsx
    templates/
      DashboardLayout.tsx
      AuthLayout.tsx
  pages/
    dashboard/
      DashboardPage.tsx
    invoices/
      InvoiceListPage.tsx
      InvoiceDetailPage.tsx
  hooks/
  services/
```

**When to use:** Design-system-first products where component reuse across features is the primary organizing concern. Works well with a dedicated design system team.

**Team size fit:** Teams where UI consistency is a primary concern; less suited to products where feature velocity is the priority.

## Option 3: Domain-Driven

Mirrors the backend domain structure. Each domain owns its UI components.

```
src/
  domains/
    billing/
      components/
      hooks/
      pages/
    invoicing/
      components/
      hooks/
      pages/
  shared/
    ui/
    hooks/
  shell/
    App.tsx
    Router.tsx
    Layout.tsx
```

**When to use:** Large products where frontend teams have domain ownership that mirrors the backend. Overkill for solo or small teams.

**Team size fit:** Medium to large teams with domain ownership.

## Recommended Default

**Feature-based structure** is a strong fit when product capabilities are the clearest ownership boundary. It matches the backend's feature-based organization (from [architecture/backend/structure.md](structure.md)), making it easy to reason about the full stack for a given feature. Keep shared UI components in `shared/components/` with clear rules about what qualifies as "shared" (used by at least two features).

## Related Documents

- [Backend structure](../backend/architecture/structure.md)
- [Frontend state management](state-management.md)
- [Module boundaries](../quality/code/module-boundaries.md)
