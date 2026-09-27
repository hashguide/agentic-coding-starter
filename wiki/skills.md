# Skills

Skills contain **workflow**, while `.docs/playbook/` contains **knowledge**.

Keeping those separate allows coding tools to load a short procedure first and progressively retrieve only the professional guidance needed for the current decision.

## Canonical skills

The canonical repository skills live in `.agents/skills/`:

- `develop-product` — turn incomplete intent into project specifications.
- `design-feature` — resolve a capability's product, IA, UX, states, and technical boundaries.
- `decide-architecture` — compare and record consequential technical choices.
- `implement-feature` — build specified behavior and keep documentation synchronized.
- `review-product` — perform cross-discipline critique and validation.

Tool-specific adapters may reference these canonical skills rather than duplicate the full workflow.

## What belongs in a skill

A skill should define:

- when the workflow applies
- which project truth to read
- how to route to relevant guidance
- what decisions the agent owns
- what output must be written
- how the work is checked before handoff

## What belongs in the playbook

The playbook should hold reusable decision knowledge such as:

- page vs drawer vs modal vs inline interaction
- task-oriented vs object-oriented IA
- polling vs SSE vs WebSocket
- synchronous vs queued execution
- SQL vs document storage
- session vs token authentication
- visual direction, typography, color, density, and brand guidance
- testing, security, reliability, observability, and deployment choices

Skills should reference this guidance rather than duplicate it.

## Agent responsibility

The skills assume the agent is the senior product engineer/technical lead. Unless the owner has already specified a choice, the agent should discover what needs deciding, research material unknowns, compare viable approaches, critique the preferred direction across applicable disciplines, record the decision, and validate the result.
