---
kind: playbook
domain: delivery
topics: [delivery, environments, feature, flags, by, environment]
authority: guidance
---

# Feature Flags by Environment

Feature flag defaults must differ by environment. A flag that is on for active development must not automatically be on in production. This document defines the recommended default policy.

## Recommended Flag Defaults by Environment

| Environment | Default flag state | Rationale |
|---|---|---|
| **Local** | On for the feature being actively developed | Fast iteration; no risk |
| **Staging** | On for features being tested | Verify the feature works end-to-end before production rollout |
| **Production** | Off until explicit rollout | Safe default; requires a deliberate action to expose to users |

## Implementation

Flags are stored in the database (see [admin/feature-flags.md](../../operations/admin/feature-flags.md)). Default values in the database should be `false` (off), and environment-specific overrides are applied via seed data:

```typescript
// prisma/seed.ts
async function seedFeatureFlags() {
  const flags = [
    { key: 'new_dashboard_v2', description: 'New dashboard layout', enabled: false },
    { key: 'ai_suggestions', description: 'AI-powered suggestions in editor', enabled: false },
  ];

  for (const flag of flags) {
    await db.featureFlag.upsert({
      where: { key: flag.key },
      update: {},
      create: flag,
    });
  }

  // In development: enable flags for active work
  if (process.env.NODE_ENV === 'development') {
    await db.featureFlag.updateMany({
      where: { key: { in: ['new_dashboard_v2'] } },
      data: { enabled: true },
    });
  }
}
```

For production rollouts, enable flags through the admin panel — not by changing seed data or environment variables. The admin panel provides a visible action and audit log entry.

## Gradual Rollout Pattern

The recommended rollout sequence for a new feature:

1. **Local development:** Flag on for the developer working on the feature
2. **Staging:** Flag on for the full staging environment; QA and team testing
3. **Production rollout begin:** Flag enabled for 5–10% of users (percentage rollout)
4. **Production expand:** Increase to 25%, 50%, 75% if no issues
5. **Full rollout:** Enable flag globally (100%) or remove the flag and ship the code unconditionally

At each step, monitor error rates and user behavior before expanding.

## Keeping Flags Clean

Feature flags that are permanently enabled are not feature flags — they are dead code paths. After a feature fully rolls out, remove the flag and the conditional code branches:

```bash
# Track flag creation dates in the description field
{ key: 'new_dashboard_v2', description: 'New dashboard layout — added 2025-01-15' }

# Quarterly audit: any flag older than 90 days that is 100% enabled should be removed
```

## Related Documents

- [Admin feature flags](../../operations/admin/feature-flags.md)
- [Environments overview](overview.md)
- [Environments local development](local-development.md)
