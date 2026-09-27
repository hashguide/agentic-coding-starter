---
kind: playbook
domain: delivery
topics: [delivery, version, control, release, strategy]
authority: guidance
---

# Release Strategy

A release is a named, versioned snapshot of the codebase that is deployed to production and communicated to users. Without a release strategy, you have deployments — code moving to production — but no historical reference points, no changelog, and no way to communicate "what changed" to users or to pin down when a regression was introduced.

## Semantic Versioning

All releases follow Semantic Versioning: `MAJOR.MINOR.PATCH`

| Segment | Increment when |
|---|---|
| `MAJOR` | A breaking change is introduced (incompatible API, changed behavior that breaks existing users) |
| `MINOR` | New functionality is added in a backward-compatible way |
| `PATCH` | A bug fix that is backward-compatible |

**Pre-release suffixes:** Use `-beta.1`, `-rc.1` for pre-release versions that are deployed to staging but not yet announced to users.

**SaaS note:** Most web SaaS products start at `0.x.x` during development and `1.0.0` at public launch. Version numbers above 1.0 are meaningful to users — increment them when there's a meaningful user-visible release, not on every deploy.

## Changelog from Conventional Commits

If commits follow the [Conventional Commits](git-workflow.md) spec, changelogs can be generated automatically.

```bash
# Install conventional-changelog-cli
npm install -D conventional-changelog-cli

# Generate changelog from the last tag to HEAD
npx conventional-changelog -p conventionalcommits -i CHANGELOG.md -s
```

The generated changelog groups commits by type:
- `feat` → **Features**
- `fix` → **Bug Fixes**
- `perf` → **Performance Improvements**
- Breaking changes → **Breaking Changes** (always at the top)

**Keep the CHANGELOG.md written for users, not developers.** After auto-generating, review the entries: rename technical identifiers to user-facing feature names, remove internal chore entries, and combine related fixes into a single user-meaningful line.

## Tag-Triggered Deploy Pipeline

Tagging a release should trigger the production deploy automatically. This ties the version number to the deployed artifact.

Example GitHub Actions workflow:

```yaml
name: Release

on:
  push:
    tags:
      - 'v[0-9]+.[0-9]+.[0-9]+'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build

      - name: Deploy to production
        run: ./scripts/deploy.sh
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}

      - name: Create GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          generate_release_notes: true
```

**Tagging workflow:**

```bash
# Create and push a new release tag
git tag -a v1.2.0 -m "Release v1.2.0: invoice reminders and CSV export"
git push origin v1.2.0
```

## Release Checklist

Before tagging a release:

- [ ] All targeted PRs are merged to main
- [ ] CI is passing on main
- [ ] Staging deploy of this commit verified
- [ ] CHANGELOG.md updated and reviewed for user clarity
- [ ] Database migrations tested on staging with realistic data volume
- [ ] Rollback plan confirmed (previous tag is the rollback target)

## Related Documents

- [Git workflow](git-workflow.md)
- [Pull requests](pull-requests.md)
- [CI/CD pipeline](../infrastructure/ci-cd.md)
- [Zero-downtime deployment](../infrastructure/vps/zero-downtime.md)
