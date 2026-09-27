---
kind: playbook
domain: delivery
topics: [delivery, version, control]
authority: guidance
---

# Version Control Overview

Version control is not a backup system — it's an audit trail, a collaboration protocol, and a deployment mechanism. Developers who treat it as a backup accumulate commits like "fix," "wip," and "final2," making the history unreadable precisely when it matters most: during an incident, a merge conflict, or an onboarding for a new contributor.

## The Cost of Poor Version Control Habits

**Lost work:** Without atomic commits and regular pushes, a corrupted working directory or a bad merge can erase hours of work. The git history is only as recoverable as your commit discipline.

**Unclear history:** A codebase where every commit is "fix" or "update" provides no signal about when a bug was introduced, who made a decision, or why a change was made. The commit message is documentation — the only documentation that stays in sync with the code automatically.

**Painful collaboration — even with future-you:** A monolithic commit that touches eight files across four features is not a unit of work. It cannot be reverted cleanly, reviewed efficiently, or cherry-picked safely. Solo builders suffer this problem too: returning to a project after three weeks, the commit history is the only map.

## What This Directory Covers

- [git-workflow.md](git-workflow.md) — Commit conventions, atomic commits, and message format
- [branching-strategy.md](branching-strategy.md) — Gitflow vs. trunk-based vs. GitHub Flow
- [github-setup.md](github-setup.md) — Repository settings, branch protection, secret scanning
- [pull-requests.md](pull-requests.md) — PR templates, review checklist, merge strategy
- [release-strategy.md](release-strategy.md) — Tagging, changelogs, semantic versioning

## Related Documents

- [CI/CD pipeline](../infrastructure/ci-cd.md)
- [Code quality overview](../../quality/code/overview.md)
