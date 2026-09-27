---
kind: playbook
domain: ai
topics: [ai-safety, trust, permissions]
authority: guidance
---
# AI Safety and Trust

Treat model output as probabilistic and untrusted. Product trust comes from bounded authority, validation, transparent state, and recoverable actions.

## Define authority
For every AI capability specify what the model may read, generate, recommend, modify, send, purchase, delete, execute, or expose. Use least privilege for tools and data access.

## Approval model
Require explicit user confirmation for consequential actions when the cost of an incorrect action is significant or difficult to reverse. Low-risk reversible actions can use narrower automatic authority when the product makes that behavior clear.

## Validate boundaries
Validate structured outputs and tool arguments before execution. Re-check authorization at the tool/server boundary. Treat retrieved content, web pages, uploaded files, and tool output as potentially hostile or misleading context.

## Communicate state
Make important distinctions visible when relevant: suggestion vs action, draft vs sent, queued vs completed, inferred vs verified, and model failure vs system failure.

## Recovery
Define timeout, retry, fallback, cancellation, partial completion, audit trail, and user correction paths. Preserve enough execution history to explain consequential actions.

## Review
Test prompt injection/tool misuse, permission escalation, sensitive-data leakage, fabricated identifiers, invalid structured output, tool failure, duplicate execution, and ambiguous user intent according to product risk.
