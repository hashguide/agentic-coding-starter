---
kind: playbook
domain: security
topics: [security, secrets]
authority: guidance
---

# Secrets Management

A secret committed to git is a compromised secret. A secret hardcoded in source is a secret shared with everyone who has access to that repository, now and in the future. Every secret must be externalized from code and managed with clear rotation and audit policies.

## Zero Hardcoded Secrets

No database passwords, API keys, JWT signing secrets, or encryption keys in source code. Not in `.env.example` with real values. Not in config files. Not in comments. The enforcement mechanism:

```bash
# .gitignore — non-negotiable entries
.env
.env.local
.env.production
.env.staging
*.key
*.pem
secrets/
```

Set up pre-commit hooks or CI scanning to reject commits that introduce credential patterns:

```bash
# Install git-secrets or trufflehog, then:
git secrets --install
git secrets --register-aws  # Add more patterns as needed
```

Any secret ever committed to git — even for one commit, even in a branch that was deleted — must be treated as compromised and rotated immediately. Git history is permanent and that commit exists in every clone and fork.

## Secrets Manager Comparison

| Approach | When to use | Trade-offs |
|---|---|---|
| **Environment variables** | Solo projects, simple deployments | Easy to set up; no audit trail; harder to rotate; secrets visible in process list on some systems |
| **Doppler** | Small teams, multiple environments | Good UI, env-var injection, audit log, automatic rotation hooks; requires external dependency |
| **HashiCorp Vault** | Security-critical products, regulated industries | Full audit trail, dynamic secrets, fine-grained access; significant operational overhead |
| **Cloud provider secrets managers** (AWS SSM/Secrets Manager, GCP Secret Manager) | Already on that cloud | Tight IAM integration; cloud-specific |

**Recommended default:** environment variables for solo/small deployments; Doppler when you need per-environment secrets, team access control, and audit history without Vault's operational overhead.

## Loading Secrets at Startup

Validate that all required secrets are present when the application starts — fail immediately, not when the first request hits a missing value:

```typescript
import { z } from 'zod';

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  STRIPE_SECRET_KEY: z.string().startsWith('sk_'),
  SENDGRID_API_KEY: z.string().startsWith('SG.'),
  ENCRYPTION_KEY: z.string().length(64), // 32 bytes hex-encoded
});

function parseEnv() {
  const result = envSchema.safeParse(process.env);
  if (!result.success) {
    console.error('❌ Missing or invalid environment variables:');
    console.error(result.error.flatten().fieldErrors);
    process.exit(1);
  }
  return result.data;
}

export const env = parseEnv();
```

See [backend/env-config.md](../backend/runtime/env-config.md) for the full env validation pattern.

## Rotation Policy

| Secret type | Rotation frequency | Notes |
|---|---|---|
| Database passwords | Every 90 days or on team member departure | Use connection pooling so rotation doesn't require restart |
| Third-party API keys | Every 90 days, or immediately on exposure | Most providers support dual active keys during rotation |
| JWT signing secrets | Every 90 days; immediately on exposure | Rotating invalidates all active sessions — plan accordingly |
| Encryption keys | Annually, or on exposure | Requires re-encrypting all data encrypted with the old key |

**Automated rotation** is preferable to manual processes for secrets that support it (AWS RDS passwords, Doppler secrets with webhooks). Manual rotation requires calendar reminders and follow-through; automated rotation does not.

**On team member departure:** rotate all secrets that the departing member had access to, regardless of your regular rotation schedule.

## Audit Trail

Know who accessed which secret and when. For environment variables alone, there is no audit trail. Doppler, Vault, and cloud secrets managers provide access logs. At minimum, log secret access attempts (not the secret values) in your application logs.

## Related Documents

- [Backend environment config](../backend/runtime/env-config.md)
- [Security checklist](checklist.md)
- [Database encryption](../data/encryption.md)
- [Observability logging](../observability/logging.md)
