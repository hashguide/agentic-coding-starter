---
kind: playbook
domain: access
topics: [access, models, model, selection]
authority: guidance
---

# Access Model Selection

The right access model matches the product type. This document provides a decision guide for the most common SaaS product patterns.

## Decision Guide

### Solo User SaaS

A product where each account is used by one person. There is no meaningful role hierarchy because there is only one user per account.

**Model:** No formal access model — the authenticated user owns all their resources.

**Implementation:** All database queries filter by `userId`. No role checks needed. If an admin panel is required, implement a separate admin role on the `users` table, checked in admin middleware.

### Team SaaS (Collaborative Product)

Multiple users share a workspace, organization, or project. Different users have different levels of permission within a shared context.

**Model:** RBAC within a tenant (organization).

**Implementation:**
- Multi-tenant data model with `organization_id` on all shared tables
- Roles table defining member roles within each organization
- Permission middleware checking role before each operation
- Row-Level Security at the database level for tenant isolation

**Reference:** [multi-tenant.md](multi-tenant.md), [rbac.md](rbac.md)

### Enterprise SaaS (Complex Org Structures)

Large customers with complex permission requirements: resource-level access, team-based visibility, approval workflows, permission inheritance.

**Model:** RBAC + ABAC hybrid, or pure ABAC.

**Implementation:**
- RBAC for coarse-grained role checks (admin, member, viewer)
- ABAC policies for resource-level conditional permissions
- May require a dedicated policy engine (OPA, Casbin) at sufficient complexity

**Reference:** [abac.md](abac.md), [rbac-vs-abac.md](rbac-vs-abac.md)

### Digital Download / One-Time Purchase

A template, plugin, tool, or asset sold once. No ongoing account required; access is granted by the purchase.

**Model:** License-based access with signed download URLs.

**Implementation:**
- Generate a license key and download links on payment confirmation
- Validate keys at download time
- Optionally implement device activation limits

**Reference:** [license-based-access.md](license-based-access.md), [entitlement-verification.md](entitlement-verification.md)

### Marketplace

A platform where multiple sellers list products and buyers purchase from them.

**Model:** Multi-tenant (sellers are tenants) + Stripe Connect (for payment splitting) + entitlement verification (buyer access to purchased items).

**Implementation:**
- Sellers have separate Stripe Connect accounts
- Buyers' purchased items are tracked in an entitlements table
- Download/access links are generated per buyer at purchase time

**Reference:** [multi-tenant.md](multi-tenant.md), [entitlement-verification.md](entitlement-verification.md), [Stripe Connect](../../integrations/providers/stripe/connect.md)

## Quick Reference Table

| Product type | Access model | Key documents |
|---|---|---|
| Solo user SaaS | userId filter only | [schema-design.md](../../data/schema-design.md) |
| Team SaaS | Multi-tenant + RBAC | [multi-tenant.md](multi-tenant.md), [rbac.md](rbac.md) |
| Enterprise SaaS | RBAC + ABAC | [abac.md](abac.md), [rbac-vs-abac.md](rbac-vs-abac.md) |
| Digital download | License-based | [license-based-access.md](license-based-access.md) |
| Marketplace | Multi-tenant + entitlement | [entitlement-verification.md](entitlement-verification.md) |

## Related Documents

- [Access models overview](overview.md)
- [Multi-tenant](multi-tenant.md)
- [RBAC](rbac.md)
- [ABAC](abac.md)
