---
name: develop-product
description: Turn a vague software idea into a coherent product definition and initial project specification. Use during discovery, early specification, or when product direction changes materially.
---

# Develop Product

1. Read `.docs/START.md`, `.docs/MANIFEST.md`, the current project brief/constraints, and execution state.
2. Use the **Idea → product definition** route in `.docs/ROUTING.md`; load only guidance needed for unresolved decisions.
3. Treat the owner's idea as intent unless they explicitly define a requirement. Establish users, problem, trigger, outcomes, alternatives, category expectations, positioning, journeys, constraints, and missing requirements.
4. Research current category behavior, competitor expectations, and material technical facts when tools are available and the decision benefits from current evidence. Record dated sources or evidence in project docs when it affects the specification.
5. Define the smallest coherent first release and identify non-functional requirements that can change architecture or experience.
6. Resolve applicable product, IA, UX, UI, branding, accessibility, architecture, data, security, operational, analytics, testing, and launch concerns from `MANIFEST.md`. Mark areas specified, deferred with rationale, or not applicable.
7. Compare and critique consequential alternatives using the relevant playbook guidance. Surface decisions that materially affect product direction, scope, cost, irreversible commitments, or explicit owner preferences for approval. Resolve routine professional decisions yourself.
8. Populate/update `.docs/project/` and create feature specs when capabilities are understood well enough to specify. Record consequential decisions as ADRs.
9. Run a cross-discipline completeness critique before handing off. Keep unresolved questions explicit and update `.docs/execution/current.md` with the next work.

Finish with project truth another agent can continue from without needing the original conversation.
