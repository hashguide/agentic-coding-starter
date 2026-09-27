---
kind: playbook
domain: ai
topics: [ai, tools, agent, permissions, approval, side-effects, tool, execution, and, approvals]
authority: guidance
---
# AI Tool Execution and Approval Gates

Give models the minimum tools and permissions needed for their role. Validate tool arguments independently of the model and authorize actions using application identity and policy.

Use human approval before actions whose consequences are expensive, destructive, externally visible, legally meaningful, financial, permission-changing, or difficult to reverse.

For autonomous tools define allowed resources, budgets, timeouts, retries, idempotency, stop conditions, audit trail and a way to cancel or contain runaway work.
