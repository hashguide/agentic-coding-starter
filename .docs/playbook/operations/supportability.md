---
kind: playbook
domain: operations
topics: [support, admin, diagnostics]
authority: guidance
---
# Supportability

Design enough operational visibility and safe tooling to understand and resolve real user problems.

## Support questions the system should answer
- Who/which tenant owns the affected resource?
- What state is the resource or job in?
- What happened recently?
- Which external dependency was involved?
- Is the problem isolated or systemic?
- Can the action be retried, repaired, refunded, reprocessed, or cancelled safely?

## Supporting capabilities
Depending on product risk, provide structured logs/correlation IDs, searchable audit events, job/run history, integration/webhook status, entitlement/billing state, error tracking, and a safe administrative surface.

## Admin actions
Administrative actions should use the same authorization and domain rules as the product where possible. Consequential actions need clear scope, confirmation where appropriate, auditability, and least privilege. Impersonation or direct data mutation requires explicit justification and strong audit controls.

## User support
Design useful user-facing errors, status/history, exports, retry paths, and self-service recovery where they reduce support load and improve trust.

## Review
Trace representative support incidents from user report to diagnosis and remediation. Confirm sensitive data is not unnecessarily exposed to operators or logs and that support tooling respects tenant boundaries.
