---
name: implement-feature
description: Implement or materially change a specified product capability while preserving project decisions, loading only relevant guidance, and keeping specifications synchronized with the code.
---

# Implement Feature

1. Read `.docs/START.md`, execution state, the relevant feature specification, acceptance criteria, and project truth governing the change.
2. Verify the specification covers the applicable feature-level areas in `.docs/MANIFEST.md`. Resolve missing routine requirements using the relevant playbook; escalate only consequential product choices.
3. Load playbook guidance only for unresolved or implementation-specific decisions through `.docs/ROUTING.md`.
4. Inspect existing code, tests, contracts, design patterns, and rendered behavior before editing. Preserve valid project conventions unless the specification or an accepted decision changes them.
5. Implement the complete applicable behavior: interaction states, errors/recovery, permissions, responsive/accessibility behavior, data integrity, performance, observability, analytics, and tests.
6. Critique the implementation against the specification and actual user journey. Inspect rendered UI and realistic states when the feature is user-facing.
7. Verify with the project's available type, lint, test, build, runtime, security, accessibility, performance, and visual-review mechanisms as applicable.
8. Update project, feature, decision, and execution docs whenever implementation changes documented behavior, contracts, architecture, data, IA, UX, or visual direction.
9. Use `review-product` for meaningful user-facing, cross-cutting, or release-critical work.

Do not trade away documented product behavior for implementation convenience without recording and resolving the decision.
