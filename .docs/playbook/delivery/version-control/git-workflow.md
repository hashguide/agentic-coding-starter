---
kind: playbook
domain: delivery
topics: [delivery, version, control, git, workflow]
authority: guidance
---

# Git Workflow

A commit is a unit of historical record. A well-formed commit is self-describing: the type tells you what kind of change it is, the scope tells you what area it affects, and the message tells you why. A reader should be able to understand the project's evolution from the commit log without opening a single file.

## Conventional Commits

The Conventional Commits specification provides a standard commit message format that enables automated changelog generation and clear history scanning.

**Format:**

```
<type>[optional scope]: <description>

[optional body]

[optional footer]
```

**Types:**

| Type | Use for |
|---|---|
| `feat` | A new feature visible to a user |
| `fix` | A bug fix |
| `chore` | Maintenance that doesn't affect user behavior (dependency bumps, config) |
| `refactor` | Code restructuring with no behavior change |
| `wiki` | Documentation only |
| `test` | Adding or fixing tests |
| `perf` | Performance improvement |
| `ci` | Changes to CI/CD pipeline configuration |
| `style` | Formatting changes (whitespace, semicolons) — never logic |

**Examples:**

```
feat(auth): add Google OAuth login
fix(invoices): prevent duplicate line items on rapid submit
refactor(notifications): extract badge count into useBadgeCount hook
chore(deps): bump stripe from 12.1.0 to 14.0.0
test(payments): add webhook signature verification integration test
```

## Atomic Commits

An atomic commit does exactly one thing. It could be reverted without also reverting unrelated changes. It passes tests on its own.

**Not atomic:**
```
feat: add notifications and fix invoice validation and update styles
```

**Atomic:**
```
feat(notifications): add notification bell and unread count badge
fix(invoices): validate line item total is non-negative
style(layout): align dashboard header buttons
```

The test for atomicity: could you `git revert` this commit in isolation and have the codebase still work? If reverting it would break something unrelated, the commit contains multiple things.

## Commit Frequency

Commit frequently during active development — after every meaningful, working increment. Do not commit broken states to a shared branch. On a feature branch, committing a failing test with a comment "wip: failing test for edge case" is acceptable if it's cleaned up before the PR merges.

Prefer more frequent, smaller commits over infrequent large ones. It's easier to squash commits on merge than to reconstruct history from a 40-file diff.

## Commit Message Quality Rules

- **Subject line:** Imperative mood, present tense. "add feature" not "added feature" or "adds feature." 72 characters or fewer.
- **Body:** Explain the why, not the what. The diff explains what changed. The body explains why this change was necessary.
- **Breaking changes:** Mark in the footer with `BREAKING CHANGE:` and a description of what callers must update.

```
feat(auth)!: switch session tokens from JWT to server-side sessions

JWT approach made token invalidation impossible on logout without a
denylist. Server-side sessions allow immediate invalidation.

BREAKING CHANGE: clients must send session cookie on every request.
The Authorization header is no longer accepted.
```

## Related Documents

- [Branching strategy](branching-strategy.md)
- [Pull requests](pull-requests.md)
- [Release strategy](release-strategy.md)
