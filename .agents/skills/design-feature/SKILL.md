---
name: design-feature
description: Define or refine a meaningful product capability before implementation, including user journey, IA placement, interaction states, technical boundaries, permissions, and acceptance criteria.
---

# Design Feature

1. Read `.docs/START.md`, `.docs/MANIFEST.md`, applicable project truth, and existing documents for the feature.
2. Route through `.docs/ROUTING.md` to the smallest relevant set of product, research, IA, interaction, visual/brand, frontend, backend, data, security, accessibility, analytics, and operational guidance.
3. Establish the user's goal and shortest coherent journey. Determine where the capability belongs in the product's information architecture before designing screens or components.
4. Research current category patterns when the feature has established market expectations or when comparison would materially improve the decision.
5. Compare viable interaction and technical approaches. Select patterns from behavior and constraints: placement, page/drawer/modal/inline treatment, save model, synchronous/asynchronous execution, optimistic/confirmed updates, realtime behavior, and state persistence where applicable.
6. Critique the proposed experience for clarity, friction, hierarchy, discoverability, responsiveness, accessibility, consistency, failure/recovery, trust, and visual fit with the product direction.
7. Resolve every applicable feature-level area in `MANIFEST.md`: states, responsive behavior, content, permissions, data/contracts, security, performance, observability, analytics, tests, support/recovery, and acceptance criteria.
8. Create/update the feature using `.docs/templates/feature/`. Update project-wide docs when the feature changes shared IA, architecture, data, access, brand/visual direction, or product behavior. Record consequential decisions in `.docs/decisions/`.

The feature specification should be sufficient for an implementation agent to build without inventing product behavior.
