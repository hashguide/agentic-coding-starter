---
name: review-product
description: Critique a product, feature, or implementation against the relevant product, IA, UX, visual, accessibility, security, performance, reliability, and release expectations. Use before completion or when quality feels weak.
---

# Review Product

1. Identify the review scope, documented user outcome, acceptance criteria, current stage, and applicable areas in `.docs/MANIFEST.md`.
2. Select only the relevant review routes from `.docs/ROUTING.md`; load project/feature truth before reusable guidance.
3. Research current category expectations when product quality, interaction patterns, visual conventions, or competitive completeness are part of the review.
4. Inspect actual behavior, code, and rendered UI when available. Exercise realistic loading, empty, populated, error, long-content, responsive, permission, interruption, and recovery states that apply.
5. Critique every applicable discipline: product usefulness/scope, journeys, IA, UX, interaction, visual/brand coherence, content, accessibility, frontend/backend architecture, data, security/privacy, performance, reliability, analytics/observability, testing, operations/support, and release readiness.
6. Compare the implementation with project truth and evidence. Identify missing requirements, friction, inconsistencies, weak hierarchy, correctness risks, and operational gaps.
7. Classify findings as blocking or improvement. Give every material finding a concrete remediation and identify the specification or decision that must change.
8. Save durable findings under `.docs/validation/`. Update project or feature truth when review reveals a requirements change.
9. Re-review remediated blocking findings before marking the work complete.

Completion means every applicable quality area has been intentionally resolved and no blocking finding remains.
