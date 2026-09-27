---
kind: playbook
domain: delivery
topics: [delivery, version, control, branching, strategy]
authority: guidance
---

# Branching Strategy

A branching strategy defines how code moves from a developer's machine to production. The wrong strategy creates either too much process (blocking fast iteration) or too little (creating merge chaos). Choose based on team size, release cadence, and how much risk you can absorb from main being broken at any given time.

## Options

### Gitflow

Gitflow uses two long-lived branches (`main` and `develop`), plus short-lived feature, release, and hotfix branches.

- `main` always reflects production
- `develop` is the integration branch; features merge here
- `release/*` branches are cut from develop when preparing a release
- `hotfix/*` branches are cut from main for emergency production fixes

**Tradeoffs:**

| Dimension | Gitflow |
|---|---|
| Complexity | High — multiple branch types, strict rules |
| Release cadence fit | Versioned releases (v1.0, v2.0) with defined release dates |
| Team size fit | Medium to large teams with separate roles (dev, QA, release manager) |
| Solo/small team fit | Poor — overhead is disproportionate to the benefit |

### GitHub Flow

GitHub Flow uses one long-lived branch (`main`) plus short-lived feature branches. Features merge directly to main via pull request and deploy immediately.

- `main` is always deployable
- Feature branches are short-lived (hours to days, not weeks)
- Every merge to main triggers a deploy

**Tradeoffs:**

| Dimension | GitHub Flow |
|---|---|
| Complexity | Low — one rule: main is always deployable |
| Release cadence fit | Continuous deployment; every PR is a release |
| Team size fit | Solo to small teams (2–8) |
| Solo/small team fit | Excellent |

### Trunk-Based Development

All developers commit directly to `main` (or use very short-lived branches, merged same-day). Feature flags control which features are visible to users.

**Tradeoffs:**

| Dimension | Trunk-Based |
|---|---|
| Complexity | Low branching complexity; higher feature flag complexity |
| Release cadence fit | High-frequency deploys, teams with strong CI/CD discipline |
| Team size fit | Small to large, but requires mature CI and feature flag infrastructure |
| Solo/small team fit | Good once feature flag infrastructure is in place |

## Recommended Default

**GitHub Flow** is a strong fit for a small team with frequent integration and deployment.

Reasoning:
- `main` is always deployable — you can respond to production issues without navigating branch state
- Short-lived feature branches keep context tight and reviews small
- No release branches means no merge back-conflicts
- Simple enough that the process doesn't become the project

**Adopt trunk-based development** when you have feature flag infrastructure in place and your team is shipping to main multiple times per day. This is a natural evolution, not a day-one choice.

**Adopt Gitflow** only if you have genuine versioned release requirements (shipping desktop software, an on-premise product) and a team large enough that release coordination is a real problem.

## Branch Naming Convention

```
feat/<short-description>
fix/<short-description>
chore/<short-description>
docs/<short-description>
```

Examples:
```
feat/google-oauth
fix/invoice-duplicate-submit
chore/bump-stripe-sdk
```

## Related Documents

- [Git workflow](git-workflow.md)
- [Pull requests](pull-requests.md)
- [GitHub setup](github-setup.md)
- [CI/CD pipeline](../infrastructure/ci-cd.md)
