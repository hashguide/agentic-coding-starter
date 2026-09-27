---
kind: agent-system
topics: [adr, decisions, tradeoffs]
---
# Decision Records

Create a decision record when a choice is expensive to reverse, affects several features, establishes a public contract, changes security/data guarantees, or is likely to be questioned later.

A decision record contains:
- context and constraints;
- viable options considered;
- the selected approach;
- why it fits this project;
- consequences and follow-up work;
- status: proposed, accepted, superseded, or rejected.

Keep routine local implementation choices in code and feature docs rather than producing an ADR for every decision.
