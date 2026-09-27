---
kind: playbook
domain: operations
topics: [operations, admin, feature, flags]
authority: guidance
---

# Admin Feature Flags

Feature flags let you enable or disable features without deploying code — per user, per plan tier, or gradually by percentage. They also allow safe rollout of in-progress work by enabling a feature for internal users before external users.

## Feature Flag Schema

```sql
-- Prisma schema
model FeatureFlag {
  id          String   @id @default(cuid())
  key         String   @unique       -- e.g., "new_dashboard_v2"
  description String
  enabled     Boolean  @default(false) -- global default
  rollout     Int      @default(0)   -- 0-100 percentage rollout
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  overrides   FeatureFlagOverride[]
}

model FeatureFlagOverride {
  id        String      @id @default(cuid())
  flag      FeatureFlag @relation(fields: [flagId], references: [id])
  flagId    String
  targetType String     -- "user" | "plan"
  targetId  String      -- userId or plan name
  enabled   Boolean
  createdAt DateTime    @default(now())

  @@unique([flagId, targetType, targetId])
}
```

## Targeting Strategies

**Per-user override:** Enable or disable a flag for a specific user regardless of the global setting. Used for beta testers, support cases, or power users.

**Per-plan tier:** Enable a flag for all users on a given subscription plan. Used for gating features by tier (e.g., `advanced_analytics` is enabled for `pro` and `enterprise`).

**Percentage rollout:** Enable a flag for a random percentage of users. Used for gradual rollouts to catch issues before a full launch.

## Flag Evaluation Logic

```typescript
export async function isFeatureEnabled(
  flagKey: string,
  userId: string,
  userPlan: string
): Promise<boolean> {
  const flag = await db.featureFlag.findUnique({
    where: { key: flagKey },
    include: {
      overrides: {
        where: {
          OR: [
            { targetType: 'user', targetId: userId },
            { targetType: 'plan', targetId: userPlan },
          ],
        },
      },
    },
  });

  if (!flag) return false;

  // Check per-user override first (highest priority)
  const userOverride = flag.overrides.find(
    (o) => o.targetType === 'user' && o.targetId === userId
  );
  if (userOverride) return userOverride.enabled;

  // Check per-plan override
  const planOverride = flag.overrides.find(
    (o) => o.targetType === 'plan' && o.targetId === userPlan
  );
  if (planOverride) return planOverride.enabled;

  // Percentage rollout: hash userId for stable, consistent results
  if (flag.rollout > 0 && flag.rollout < 100) {
    const hash = parseInt(
      createHash('md5').update(`${flagKey}:${userId}`).digest('hex').slice(0, 8),
      16
    );
    return (hash % 100) < flag.rollout;
  }

  return flag.enabled;
}
```

## Admin API Endpoints

```typescript
// GET /admin/feature-flags
app.get('/admin/feature-flags', requireAdmin, async (req, res) => {
  const flags = await db.featureFlag.findMany({ include: { overrides: true } });
  res.json(flags);
});

// PATCH /admin/feature-flags/:key
app.patch('/admin/feature-flags/:key', requireAdmin, async (req, res) => {
  const { enabled, rollout } = z
    .object({ enabled: z.boolean().optional(), rollout: z.number().int().min(0).max(100).optional() })
    .parse(req.body);

  const flag = await db.featureFlag.update({
    where: { key: req.params.key },
    data: { ...(enabled !== undefined && { enabled }), ...(rollout !== undefined && { rollout }) },
  });

  await writeAuditLog({
    actorId: req.session.userId!,
    action: 'feature_flag.updated',
    targetId: flag.id,
    metadata: { key: flag.key, changes: req.body },
  });

  res.json(flag);
});

// POST /admin/feature-flags/:key/overrides
app.post('/admin/feature-flags/:key/overrides', requireAdmin, async (req, res) => {
  const { targetType, targetId, enabled } = z
    .object({
      targetType: z.enum(['user', 'plan']),
      targetId: z.string(),
      enabled: z.boolean(),
    })
    .parse(req.body);

  const flag = await db.featureFlag.findUnique({ where: { key: req.params.key } });
  if (!flag) return res.status(404).json({ error: 'Flag not found' });

  await db.featureFlagOverride.upsert({
    where: { flagId_targetType_targetId: { flagId: flag.id, targetType, targetId } },
    create: { flagId: flag.id, targetType, targetId, enabled },
    update: { enabled },
  });

  await writeAuditLog({
    actorId: req.session.userId!,
    action: 'feature_flag.override.set',
    targetId: flag.id,
    metadata: { key: flag.key, targetType, targetId, enabled },
  });

  res.json({ success: true });
});
```

## Environment Defaults

Feature flag defaults should differ across environments. See [feature flags by environment](../../delivery/environments/feature-flags-by-environment.md) for the recommended defaults table.

## Related Documents

- [Admin overview](overview.md)
- [Admin audit log](audit-log.md)
- [Feature flags by environment](../../delivery/environments/feature-flags-by-environment.md)
