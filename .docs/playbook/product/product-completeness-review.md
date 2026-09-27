---
kind: playbook
domain: product
topics: [product-review, completeness]
authority: guidance
---
# Product Completeness Review

Use `.docs/MANIFEST.md` as the coverage model and review the product at the depth appropriate to its risk and stage.

## Product
Verify target user, problem, trigger, desired outcome, value proposition, scope, positioning, category baseline, activation/retention logic, and explicit constraints are coherent.

## Experience
Verify primary journeys, IA, navigation, interaction models, feature states, responsive behavior, accessibility, content/terminology, brand/visual direction, onboarding, and recovery behavior are intentionally defined.

## System
Verify architecture supports the promised experience: frontend/backend boundaries, data model/lifecycle, authentication/authorization, async/realtime work, integrations, caching/search, concurrency, performance, and non-functional requirements where applicable.

## Trust and operations
Verify validation, security/privacy, abuse controls, audit needs, observability, product analytics, support/admin needs, reliability/recovery, testing, environments, deployment, migrations, and launch criteria.

## Review method
Inspect project truth, representative feature specs, code/runtime behavior, rendered screens, and current category expectations where relevant. Mark each applicable area specified, deferred with rationale, or not applicable. Identify contradictions between documents and implementation.

## Completion
A completeness review passes when the first-release user journeys can be delivered coherently, applicable quality areas have intentional answers, project truth matches the intended implementation, and blocking findings have owners/remediation.
