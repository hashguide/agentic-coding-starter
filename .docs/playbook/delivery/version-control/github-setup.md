---
kind: playbook
domain: delivery
topics: [delivery, version, control, github, setup]
authority: guidance
---

# GitHub Repository Setup

Repository settings are security configuration. Default settings on a new GitHub repo allow direct pushes to main, accept unreviewed code, and do not scan for accidentally committed secrets. Every item in this checklist must be configured before any code is pushed to the repository.

## Repository Setup Checklist

### Branch Protection — main

Go to **Settings → Branches → Add rule**, target `main`:

- [ ] **Require a pull request before merging** — enabled
  - [ ] Required approvals: at least 1 (for solo builders: self-review via a brief PR description before merging)
  - [ ] **Dismiss stale pull request approvals when new commits are pushed** — enabled
- [ ] **Require status checks to pass before merging** — enabled
  - Add every CI check (type check, lint, tests) as a required status check
  - [ ] **Require branches to be up to date before merging** — enabled
- [ ] **Require signed commits** — enabled (see signed commits section below)
- [ ] **Do not allow bypassing the above settings** — enabled
  - This applies the rules even to repository administrators

### Secret Scanning

Go to **Settings → Security → Code security and analysis**:

- [ ] **Secret scanning** — enabled
- [ ] **Push protection** — enabled (blocks pushes that contain known secret patterns)
- [ ] **Dependabot security updates** — enabled (automatically opens PRs for vulnerable dependencies)
- [ ] **Dependabot version updates** — optional but recommended (keeps dependencies current)

### Repository Visibility

- [ ] Confirm repository visibility matches intent (private for most SaaS source code)
- [ ] Remove default `main` branch write access for outside collaborators if any exist

## Signed Commits

Signed commits prove that a commit was made by a specific key holder, preventing commit author spoofing.

Setup (using GPG):

```bash
# Generate a GPG key
gpg --full-generate-key

# List keys to get the key ID
gpg --list-secret-keys --keyid-format LONG

# Export the public key (paste into GitHub → Settings → SSH and GPG keys)
gpg --armor --export YOUR_KEY_ID

# Configure git to sign commits automatically
git config --global user.signingkey YOUR_KEY_ID
git config --global commit.gpgsign true
```

Alternatively, use SSH commit signing (available in GitHub since 2022 — simpler key management for most developers).

## Security

Branch protection rules and signed commits must be configured before the first code push. Secret scanning with push protection must be active before any API keys, credentials, or secrets exist in the codebase.

A secret that is committed to git — even for a single commit, even if immediately removed — must be treated as compromised. GitHub's secret scanning will still detect it in history. Rotate the credential immediately. See [security/secrets.md](../../security/secrets.md).

## Post-Setup Verification

After configuration, verify by attempting a direct push to main from the command line. The push should be rejected with a protection rule error.

```bash
git checkout main
echo "test" >> README.md
git add README.md && git commit -m "test: verify branch protection"
git push origin main
# Expected: rejected — protected branch
git reset --soft HEAD~1  # clean up the test commit
```

## Related Documents

- [Branching strategy](branching-strategy.md)
- [Pull requests](pull-requests.md)
- [Secrets management](../../security/secrets.md)
- [CI/CD pipeline](../infrastructure/ci-cd.md)
