---
kind: agent-system
topics: [context, routing, retrieval, context-budget]
---
# Context Routing

Load context by decision, not by repository size.

## Routing sequence

1. Read `START.md` and `ROUTING.md`.
2. Read the smallest relevant project documents.
3. Read the relevant feature specification when one exists.
4. Load playbook documents that cover unresolved decisions.
5. Follow related documents only when a dependency becomes material.
6. Stop loading guidance once enough information exists to make and record the decision.

## Context priority

Prefer:
1. explicit current constraints;
2. accepted project truth;
3. current feature truth;
4. decision records;
5. reusable playbook guidance;
6. implementation details.

When the correct playbook paths are unclear, classify the task with `../ROUTING.md` or use the matching repository skill under `../../.agents/skills/`.
