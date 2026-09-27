# How It Works

The scaffold separates reusable professional guidance from decisions about the product being built.

## The roles

The human owner provides the idea, constraints, preferences, domain knowledge, and approvals. Coding agents act as the senior product engineers and technical leads. They are responsible for discovering missing requirements, researching material unknowns, comparing options, designing the product and system, implementing it, and critiquing the result.

Explicit owner decisions become project constraints. Agents surface choices that materially change product direction, scope, cost, risk, or difficult-to-reverse commitments. Routine professional decisions should be resolved from evidence, project constraints, and the playbook.

## Knowledge layers

- `.docs/playbook/` — reusable decision guidance: options, selection criteria, tradeoffs, failure modes, and review questions.
- `.docs/project/` — authoritative project-wide requirements and decisions.
- `.docs/features/` — authoritative capability-level specifications.
- `.docs/decisions/` — durable records for consequential choices.
- `.docs/execution/` — current work, roadmap, and risks.
- `.docs/validation/` — review findings and evidence.
- `wiki/` — human-facing instructions for using the scaffold.

## The loop

1. Start with the owner's intent.
2. Route to the smallest relevant playbook guidance.
3. Research facts and current category expectations when they affect the decision.
4. Use `.docs/MANIFEST.md` to identify which disciplines apply.
5. Compare viable approaches and critique the preferred direction.
6. Record the chosen project behavior in project, feature, or decision docs.
7. Implement from project truth.
8. Review the actual result across applicable quality areas.
9. Resolve blocking findings and synchronize the docs.

The playbook is reference knowledge, not project truth. Once a decision is made, later agents should read the recorded project decision rather than repeatedly re-deriving it.
