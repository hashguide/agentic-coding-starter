---
kind: playbook
domain: access
topics: [access, models, single, tenant]
authority: guidance
---

# Single-Tenant Architecture

Single-tenant architecture provisions a dedicated infrastructure instance (application server, database, and associated services) for each customer. Customers are isolated by infrastructure — they never share compute or storage with other customers.

## When Single-Tenant Is Justified

Single-tenant is expensive to operate relative to multi-tenant. It is justified when the isolation requirement is contractual or regulatory — when a customer's security team or legal requirements mandate it, not when a developer believes it "might be better."

**Scenarios that justify single-tenant:**

**Data residency requirements:** A customer requires that their data never leaves a specific geographic region (EU-only, US-only, specific country). Multi-tenant shared infrastructure may not satisfy these requirements if other tenants are in other regions.

**Strict isolation contracts:** Enterprise customers in regulated industries (healthcare, finance, government) may contractually require that no other organization's data exists on the same infrastructure. "We cannot be on a shared database" is a real procurement requirement.

**Independent compliance certifications:** A customer may require their own audit trail, their own security controls, or their own compliance certification (SOC 2 per-tenant, for example) that is incompatible with shared infrastructure.

## Operational Trade-offs

| Dimension | Single-Tenant | Multi-Tenant |
|---|---|---|
| Isolation | Maximum — dedicated infrastructure | Moderate–Strong — shared infrastructure |
| Operational cost | High — N instances for N customers | Low — one instance for all customers |
| Deployment complexity | High — provisioning, migration, monitoring per customer | Low — one deployment |
| Pricing | Higher price points justify the cost | Standard SaaS pricing |
| Customer onboarding | Slower — provisioning takes time | Instant |

Single-tenant is viable as a high-touch enterprise tier where customers pay significantly more and accept longer provisioning timelines. It is not viable as the default offering for self-serve customers.

## Implementation Approach

When offering single-tenant alongside a standard multi-tenant product:

1. **Infrastructure as Code is mandatory.** Manual instance provisioning at any scale is untenable. Use Terraform, Pulumi, or SST to define the per-tenant infrastructure template and provision programmatically.

2. **Separate CI/CD pipelines.** Each customer instance may be on a different update schedule. A change deployed to one instance must not automatically deploy to all others.

3. **Consolidated monitoring.** A single monitoring dashboard aggregating health metrics across all customer instances is necessary — otherwise you're managing N separate monitoring setups.

4. **Migration tooling.** Applying a schema migration to 50 customer databases requires automation. A migration script that iterates all customer connection strings and applies the migration is the minimum.

## Related Documents

- [Multi-tenant](multi-tenant.md)
- [Access model selection](access-model-selection.md)
- [Infrastructure as Code](../../delivery/infrastructure/iac.md)
