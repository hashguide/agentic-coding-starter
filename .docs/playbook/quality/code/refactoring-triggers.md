---
kind: playbook
domain: quality
topics: [quality, code, refactoring, triggers]
authority: guidance
---

# Refactoring Triggers

Refactoring is not a project that happens "someday." It's a continuous practice applied in response to specific signals. The goal is to prevent decay from compounding, not to achieve theoretical purity. Two principles govern when to refactor: the Rule of Three tells you when to abstract, and the Boy Scout Rule tells you how much to improve at any given moment.

## The Rule of Three

**Don't abstract until something repeats a third time.**

The first time you write something, write it inline. The second time you encounter a similar need, you feel the temptation to abstract — resist. Write it inline again. The third time, you have enough information to know what the real abstraction is.

**Why the rule exists:** The first two occurrences rarely reveal the full shape of the problem. Abstracting too early creates the wrong abstraction — one that fits the first two cases but fights the third. Wrong abstractions are worse than duplications because they add indirection without adding clarity.

```typescript
// First occurrence: inline
const welcomeSubject = `Welcome to ${appName}, ${user.firstName}!`;

// Second occurrence: still inline (resist abstraction)
const resetSubject = `Reset your ${appName} password`;

// Third occurrence: now the pattern is clear
// → extract emailSubject(template: string, vars: Record<string, string>) helper
```

**When the Rule of Three doesn't apply:** For security constraints (input validation, output encoding, auth checks), never duplicate — the cost of one instance being inconsistently applied is too high. Abstract these on the first occurrence.

## The Boy Scout Rule

**Leave every file slightly better than you found it.**

When you're editing a file for a specific task, apply small improvements in passing:
- Rename a confusingly-named variable
- Extract a three-line inline comment into a well-named function
- Remove an obvious dead code path
- Add a missing type annotation

These improvements are tiny individually and require no separate PR or planning. Accumulated over hundreds of edits, they prevent the gradual decay described in [code-quality/overview.md](overview.md).

**Scope the Boy Scout Rule:** Apply it to the immediate area of the file you're working in. Do not embark on a cross-file cleanup while in the middle of a feature PR. If you notice a problem in a different file, open a ticket or create a chore branch — don't expand the current PR's scope.

## Balancing Refactoring Against Shipping Velocity

**Refactoring is the accelerator, not the brake.** Every meaningful refactor reduces the time cost of the next feature in the affected area. Skipping it defers the cost — it doesn't eliminate it.

**Signals that refactoring is overdue and affecting velocity:**
- Adding a new feature requires changing more than three files in the area
- Writing a test for an existing function requires significant mocking because the function does too many things
- Two different developers produce conflicting implementations of the same behavior in the same week
- An AI agent consistently produces code that contradicts the pattern in the area

**When to leave code alone:**
- It works, it's not changing, and it's not in a hot path of active development
- The refactor would require a large migration with no user-visible benefit
- The current implementation has a non-obvious constraint that the "cleaner" version would break (document this with a comment instead)

## Related Documents

- [Code smells](code-smells.md)
- [Dead code and cleanup](dead-code-and-cleanup.md)
- [Engineering principles](../../product/engineering-principles.md)
