---
kind: playbook
domain: access
topics: [access, models]
authority: guidance
---

# Access Models Overview

Access models answer a different question than authentication. Authentication asks "who is this user?" Access models ask "what is this user allowed to do, and on which resources?" The distinction matters: two products can use the same authentication mechanism (e.g., email + password) but have radically different access models.

## Authentication vs. Authorization

**Authentication** (proving identity): Handled by the mechanisms in [auth/options.md](../authentication/options.md) — sessions, JWTs, OAuth.

**Authorization** (determining permission shape): Handled by the access model. The access model is not about the implementation of permission checks — it's about the structure of the permission system: who can access what kind of resource under what conditions.

## Access Model Decision

The choice of access model is driven by the product type:

| Product type | Typical access model |
|---|---|
| Solo user SaaS (single user per account) | No meaningful access model — the authenticated user owns everything |
| Team SaaS (multiple users share resources) | RBAC within a tenant — roles determine what members can do |
| Enterprise SaaS (complex org structures, resource policies) | ABAC or RBAC + ABAC hybrid |
| Digital download / one-time purchase | License-based access |
| Marketplace (multiple sellers, buyers) | Stripe Connect + entitlement verification |

## What This Directory Covers

- [multi-tenant.md](multi-tenant.md) — Tenancy models: shared schema, schema-per-tenant, database-per-tenant
- [single-tenant.md](single-tenant.md) — Dedicated instances per customer
- [rbac.md](rbac.md) — Role-based access control as an access model
- [abac.md](abac.md) — Attribute-based access control — policy-driven, conditional
- [rbac-vs-abac.md](rbac-vs-abac.md) — Deciding between and combining them
- [license-based-access.md](license-based-access.md) — Digital products, license keys
- [entitlement-verification.md](entitlement-verification.md) — Post-purchase access verification
- [access-model-selection.md](access-model-selection.md) — Matching access model to product type

## Related Documents

- [Auth options](../authentication/options.md)
- [Auth RBAC](../authentication/rbac.md)
- [Database schema design](../../data/schema-design.md)
