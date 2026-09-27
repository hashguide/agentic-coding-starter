---
kind: playbook
domain: security
topics: [security]
authority: guidance
---

# Security Overview

Security in a SaaS product is not a single feature or a final checklist — it is a layered posture applied consistently across every part of the system. This document introduces the mindset and the threat modeling exercise used throughout this library.

## Defence in Depth

No single control should be the only barrier between an attacker and your data. Defence in depth means redundant controls at every layer: even if one layer is bypassed, the next layer catches the attack.

| Layer | Primary control | Backup if primary fails |
|---|---|---|
| Transport | TLS everywhere | HSTS prevents protocol downgrade |
| Auth | Password hashing + sessions | Rate limits + MFA |
| Input | Schema validation at boundary | Parameterized queries in ORM |
| Database | RLS policies | Application-level userId filters |
| Secrets | Env vars / secrets manager | Rotation + audit logs |
| Frontend | Output encoding | Content Security Policy |
| Infrastructure | Firewall rules | Intrusion detection |

If you find yourself saying "we're safe because of X," ask: what happens if X fails? The answer should be "Y catches it."

## Threat Modeling

A threat model does not require a formal framework. Before building any feature that touches user data or auth, answer four questions:

**1. What are we protecting?**
Be specific: user PII, billing data, API keys, session tokens, uploaded files.

**2. From whom?**
List concrete threat actors: external attackers over the internet, compromised third-party dependencies, malicious authenticated users, insiders with database access.

**3. What's the impact if it fails?**
Rate each threat: data leak affecting all users (critical), a single account taken over (high), session fixation in edge-case browser (medium).

**4. What controls are in place?**
Map each threat to the controls that reduce its likelihood and the controls that limit its blast radius if it succeeds anyway.

This exercise takes 30 minutes per feature and prevents the mistakes that take months to clean up.

## How Security Notes Are Organized

Every document in this library that covers a security-relevant topic contains a Security section. This overview connects them:

- **Transport:** [security/transport.md](transport.md) — TLS, HTTPS, HSTS, security headers
- **Input validation:** [security/input-validation.md](input-validation.md) — schema validation, XSS, SQLi, file uploads
- **Secrets:** [security/secrets.md](secrets.md) — no hardcoded secrets, rotation, audit
- **Data privacy:** [security/data-privacy.md](data-privacy.md) — minimization, export, erasure
- **Auth:** [auth/session-management.md](../access/authentication/session-management.md) — session cookies, JWT, rotation
- **Access control:** [access-models/overview.md](../access/models/overview.md) — RBAC, ABAC, multi-tenancy
- **Database:** [database/schema-design.md](../data/schema-design.md) — RLS, soft/hard delete
- **Pre-launch:** [security/checklist.md](checklist.md) — per-layer sign-off

## Related Documents

- [Security checklist](checklist.md)
- [Auth options](../access/authentication/options.md)
- [Database encryption](../data/encryption.md)
- [Compliance overview](../compliance/overview.md)
