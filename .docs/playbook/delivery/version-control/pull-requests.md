---
kind: playbook
domain: delivery
topics: [delivery, version, control, pull, requests]
authority: guidance
---

# Pull Requests

A pull request is a unit of review, not a dump of work. The goal is for any reviewer — or your future self — to understand what changed, why, and whether it's correct, in a single sitting. PRs that fail this standard have a consistent cause: they're too large, too vague, or both.

## PR Template

Add this to `.github/PULL_REQUEST_TEMPLATE.md` in the repository:

```markdown
## What

[One paragraph: what does this PR do? What user-visible behavior changes?]

## Why

[One paragraph: what problem does this solve, or what requirement does it implement? Link to the issue/ticket if one exists.]

## How

[Optional: if the approach is non-obvious, explain the key design decisions. Skip if the diff is self-explanatory.]

## Checklist

- [ ] Follows the architecture doc for this area (link if relevant)
- [ ] Tests included (unit and/or integration as appropriate)
- [ ] Scope is contained — no unrelated changes bundled in
- [ ] No raw SQL strings (ORM query builder only)
- [ ] No hardcoded secrets
- [ ] No raw server error text exposed to the client
- [ ] Migration included if schema changed
```

## Review Checklist

When reviewing a PR (yours or someone else's):

**Does it match the architecture?** Check if the change follows the patterns established in the relevant playbook document (e.g., backend API changes should follow [api-design.md](../../backend/architecture/api-design.md), auth changes should follow [auth/options.md](../../access/authentication/options.md)).

**Are tests present and meaningful?** Tests that only test the happy path and mock everything provide false confidence. Look for tests that cover the failure cases and at least one integration test that exercises the real data layer.

**Is scope contained?** A PR that fixes a bug but also "cleans up" three unrelated files, adds a dependency, and changes a UI color is three PRs. Identify and call out scope creep.

**Does it introduce security issues?** Raw SQL, secrets in code, client-exposed error details, and missing input validation are common vectors. See the security checklist in [security/checklist.md](../../security/checklist.md).

## Ideal PR Size

A PR should be reviewable in one sitting — 30 minutes or less. In practice, that means:
- Fewer than 400 lines changed (excluding generated files and lock files)
- Changes concentrated in one domain (don't mix auth changes with billing changes)
- One logical unit of work

If a feature requires more work than this, split it:
1. Schema migration + model layer (no UI, no API)
2. API endpoint (no UI)
3. UI

Each part is independently reviewable and the feature is gated behind a feature flag until all three merge.

## Merge Strategy Comparison

| Strategy | History shape | Use case |
|---|---|---|
| **Merge commit** | Preserves all commits + merge commit | Useful when individual commits are meaningful (well-formed conventional commits) |
| **Squash and merge** | Flattens all commits into one | Useful when branch commits are messy WIP; creates clean one-commit-per-feature history |
| **Rebase and merge** | Linear history, no merge commit | Clearest history but requires well-formed individual commits and no merge conflicts |

**Recommended default:** Squash and merge. It produces a clean main branch history where each entry corresponds to one PR/feature, and it doesn't require developers to maintain perfect commit discipline on feature branches.

If you're using Conventional Commits throughout development, merge commit or rebase preserves that structure on main — useful for automated changelog generation.

## Related Documents

- [Git workflow](git-workflow.md)
- [Branching strategy](branching-strategy.md)
- [GitHub setup](github-setup.md)
- [Release strategy](release-strategy.md)
