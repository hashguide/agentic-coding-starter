---
kind: playbook
domain: ai
topics: [ai, agent, workflow, planner, executor, validator, state-machine, workflows]
authority: guidance
---
# AI Agent Workflows

Use an agentic loop when the task genuinely requires iterative decisions, tool use, changing state, or recovery from intermediate results.

Use a deterministic workflow when the steps and branching rules are known. Deterministic orchestration is easier to test, constrain and operate.

For agentic workflows define goal, state, available tools, permissions, maximum steps/cost/time, checkpoints, human approval gates, failure recovery, resumability, event/audit history and success criteria.

Separating planning, execution and validation can improve reliability when each role has a clear contract; keep them combined when the task is simple enough that separation adds only overhead.
