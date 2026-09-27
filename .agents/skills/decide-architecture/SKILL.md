---
name: decide-architecture
description: Evaluate and record a consequential technical choice using product constraints and relevant tradeoffs. Use for system shape, data, APIs, async work, access, integrations, infrastructure, or other durable architecture decisions.
---

# Decide Architecture

1. Read the product/feature requirements that create the technical need plus relevant constraints and non-functional requirements.
2. Route through `.docs/ROUTING.md` to the smallest applicable architecture, backend, frontend, data, security, reliability, delivery, or integration guidance.
3. Research current platform/library/service capabilities when version, support, limits, security, or operational characteristics materially affect the choice.
4. Identify viable approaches and compare them against documented behavior, scale, consistency, reliability, security, privacy, performance, operational burden, cost, team/tooling, migration risk, and likely change pressure.
5. Critique the preferred option for failure modes, concurrency, observability, recovery, maintainability, user-experience consequences, and security boundaries that apply.
6. Choose the simplest architecture that satisfies the requirements and preserves sensible extension points where change is plausible.
7. Update authoritative project or feature technical design. Create an ADR when the decision is cross-cutting, expensive to reverse, or important to understand later.
8. Record implementation consequences, migrations, failure/recovery behavior, validation requirements, and remaining risks.

Architecture exists to support the product specification and user experience.
