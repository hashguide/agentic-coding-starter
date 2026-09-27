---
kind: agent-system
topics: [multi-agent, coordination, parallel-work]
---
# Multi-Agent Coordination

Parallel work is useful when tasks have clear boundaries and shared contracts are stable.

## Good parallel boundaries
- independent research or critique;
- separate features with no shared schema changes;
- frontend and backend work after the API/event contract is accepted;
- implementation and independent review;
- tests for stable behavior.

## Coordinate before parallel work
Record shared contracts first: data shapes, API/event schemas, ownership boundaries, naming, migration order and acceptance criteria.

## Integration owner
One agent or task must own integration: resolve conflicting assumptions, run combined checks, update project truth, and verify the complete user journey.
