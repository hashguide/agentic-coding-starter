# Workflows

The repository provides five small agent skills. They describe repeatable workflows while the detailed knowledge stays in `.docs/playbook/`.

## Develop product

Use when the starting point is an idea, incomplete requirements, or a material product-direction change.

The agent researches the category when useful, establishes users and outcomes, resolves the applicable manifest areas, compares consequential choices, and writes authoritative project truth.

## Design feature

Use before implementing a meaningful capability.

The agent determines the feature's place in the product, user journey, IA, interaction model, states, visual/brand fit, accessibility, technical boundaries, permissions, analytics, failure/recovery behavior, and acceptance criteria. It critiques the experience before implementation.

## Decide architecture

Use for durable technical choices.

The agent compares viable approaches against actual product behavior, non-functional requirements, reliability, security, performance, cost, operations, and change pressure. Current platform or provider facts should be researched when they materially affect the decision.

## Implement feature

Use once product behavior is sufficiently specified.

The agent checks specification completeness, implements the whole applicable behavior, verifies code and rendered states, and keeps project truth synchronized.

## Review product

Use for critique, pre-completion checks, and release readiness.

The agent reviews every applicable discipline in `.docs/MANIFEST.md`, including product completeness, journeys, IA, UX, UI/brand coherence, accessibility, architecture, data, security, performance, reliability, analytics/observability, testing, operations, support, and release readiness.

## Handoffs

A typical lifecycle is:

`develop-product → design-feature → decide-architecture (when needed) → implement-feature → review-product`

Skills can be entered independently when the project is already farther along. They should read existing project truth before revisiting decisions.
