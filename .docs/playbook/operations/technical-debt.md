---
kind: playbook
domain: operations
topics: [operations, technical, debt]
authority: guidance
---

# Technical Debt

Technical debt is the accumulated cost of shortcuts taken under pressure. Some debt is intentional and acceptable — a deliberate shortcut now that will be cleaned up later. Unintentional or untracked debt is a tax on every future change.

## The Boy Scout Rule (Operational Context)

Leave the code better than you found it. When you touch a file for a bug fix or feature, fix one adjacent thing that's clearly wrong — a confusing variable name, an unnecessary duplication, a missing type annotation. Do not let this expand into a full refactor; one small improvement per touch.

The Boy Scout Rule works at the pull request level: the PR is allowed to leave the codebase incrementally better than it found it, not significantly better. Significant refactors are separate PRs.

## Time Allocation

Allocate a fixed percentage of engineering capacity to debt paydown:

**Recommended: 20% of sprint capacity**

This means if a two-week sprint has 10 developer-days, 2 developer-days go to tracked debt items — not to new features, not to "we'll get to it soon."

The 20% rule works because:
- It's predictable (stakeholders know to expect it)
- It prevents debt from accumulating faster than it's paid down
- It creates a natural pressure to prioritize the most impactful items

Adjust based on product stage: a very early pre-launch product may use 5% (debt is acceptable while finding product-market fit); a scaling product with a growing team may need 30–40% to stabilize the codebase.

## Tracking Debt

Every workaround introduced under deadline pressure gets a tracked ticket immediately.

**What to track:**
- Known bugs that are not immediately blocking but will become more expensive to fix later
- Architectural shortcuts (direct DB access in a route that should use a service layer)
- Missing tests for critical paths
- Performance bottlenecks that are not yet causing issues but will
- Third-party library upgrades delayed past their end of security support

**Where to track:** Same issue tracker as everything else (Linear, GitHub Issues, Jira). Do not put debt in a separate "someday" document — it disappears.

```markdown
# Debt ticket template

Title: [DEBT] One-line description of the problem

**Current behavior:** What the code does now that's problematic.

**Why it's debt:** What specific future work this makes harder or riskier.

**Proposed fix:** The right long-term solution.

**Effort estimate:** S / M / L

**Added:** 2025-01-15 (after MVP launch, deferred to stabilize)
```

## What Is Not Debt

Not everything imperfect is debt worth tracking:
- A simple function that could be slightly cleaner but works correctly
- A well-understood abstraction that could theoretically be more general
- Documentation that could be more thorough

Debt is an obligation — something that will incur interest (slowing down future work) if not paid down. Over-tracking creates noise that drowns out real debt.

## Refactor Discipline

Refactoring without a test suite is risky. Before refactoring:
1. Write tests that document current behavior
2. Refactor
3. Verify tests still pass

A refactor that changes behavior is not a refactor — it's a bug.

Never refactor and add features in the same PR. Mixing concerns makes the diff unreadable and makes it impossible to identify which change introduced a regression.

## Related Documents

- [Testing strategy](../quality/testing/strategy.md)
- [Code quality overview](../quality/code/overview.md)
- [Version control branching strategy](../delivery/version-control/branching-strategy.md)
