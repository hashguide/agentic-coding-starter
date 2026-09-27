---
kind: playbook
domain: operations
topics: [operations, admin, user, management]
authority: guidance
---

# Admin User Management

The user management panel is the primary support tool: finding accounts, understanding their state, resolving issues, and taking corrective actions. Every action taken here must be logged.

## User List and Search

```typescript
// GET /admin/users?q=<search>&page=1
app.get('/admin/users', requireAdmin, async (req, res) => {
  const { q, page = '1' } = req.query as { q?: string; page?: string };
  const pageNum = Math.max(1, parseInt(page));
  const pageSize = 25;

  const where = q
    ? {
        OR: [
          { email: { contains: q, mode: 'insensitive' as const } },
          { name: { contains: q, mode: 'insensitive' as const } },
        ],
      }
    : {};

  const [users, total] = await Promise.all([
    db.user.findMany({
      where,
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
        subscriptionStatus: true,
        createdAt: true,
        suspendedAt: true,
      },
      orderBy: { createdAt: 'desc' },
      skip: (pageNum - 1) * pageSize,
      take: pageSize,
    }),
    db.user.count({ where }),
  ]);

  res.json({ users, total, page: pageNum, pageSize });
});
```

## User Impersonation

Impersonation allows an admin to see the product exactly as a specific user does — essential for debugging support issues.

**Requirements:**
- A persistent, undismissable banner must be shown during the entire impersonation session
- The session must be time-boxed (expire automatically)
- Every action taken while impersonating is logged against both the admin actor and the impersonated user

```typescript
// POST /admin/users/:userId/impersonate
app.post('/admin/users/:userId/impersonate', requireAdmin, async (req, res) => {
  const adminId = req.session.userId!;
  const targetUserId = req.params.userId;

  const targetUser = await db.user.findUnique({ where: { id: targetUserId } });
  if (!targetUser) return res.status(404).json({ error: 'User not found' });

  await writeAuditLog({
    actorId: adminId,
    action: 'user.impersonate.start',
    targetId: targetUserId,
    metadata: {},
  });

  // Save original admin identity so we can restore it
  req.session.impersonating = {
    adminId,
    userId: targetUserId,
    expiresAt: new Date(Date.now() + 60 * 60 * 1000), // 1 hour
  };
  req.session.userId = targetUserId;

  res.json({ success: true });
});

// DELETE /admin/impersonate — end impersonation session
app.delete('/admin/impersonate', async (req, res) => {
  if (!req.session.impersonating) {
    return res.status(400).json({ error: 'Not impersonating' });
  }

  const { adminId, userId } = req.session.impersonating;

  await writeAuditLog({
    actorId: adminId,
    action: 'user.impersonate.end',
    targetId: userId,
    metadata: {},
  });

  req.session.userId = adminId;
  delete req.session.impersonating;

  res.json({ success: true });
});
```

**Frontend — impersonation banner (React):**

```tsx
// Layout.tsx — render in every page when impersonating
function ImpersonationBanner() {
  const { impersonating } = useSession();
  if (!impersonating) return null;

  return (
    <div
      role="alert"
      style={{
        position: 'fixed',
        top: 0,
        left: 0,
        right: 0,
        zIndex: 9999,
        background: '#d97706',
        color: '#fff',
        padding: '8px 16px',
        textAlign: 'center',
        fontWeight: 600,
      }}
    >
      You are impersonating {impersonating.targetEmail}.{' '}
      <button onClick={endImpersonation}>End session</button>
    </div>
  );
}
```

The banner must be:
- Fixed position, above all other content
- Non-dismissable (no close button)
- Displayed on every page while the impersonation session is active
- Visually distinct — use a warning color, not the product's normal UI colors

## Suspension and Account Actions

```typescript
// POST /admin/users/:userId/suspend
app.post('/admin/users/:userId/suspend', requireAdmin, async (req, res) => {
  const { reason } = z.object({ reason: z.string().min(1) }).parse(req.body);
  const adminId = req.session.userId!;

  const before = await db.user.findUnique({ where: { id: req.params.userId } });

  await db.user.update({
    where: { id: req.params.userId },
    data: { suspendedAt: new Date(), suspendedReason: reason },
  });

  await writeAuditLog({
    actorId: adminId,
    action: 'user.suspend',
    targetId: req.params.userId,
    metadata: { reason, before: { suspendedAt: before?.suspendedAt } },
  });

  res.json({ success: true });
});
```

## Security

Impersonation must be unmistakably visible (persistent banner that cannot be dismissed), time-boxed, and every action taken while impersonating is logged against both the admin actor and the impersonated user.

**Why both actors?** During an impersonation session, actions appear to come from the target user. Without dual logging, it becomes impossible to distinguish a support session from unauthorized access or a compromised admin account.

## Related Documents

- [Admin overview](overview.md)
- [Admin audit log](audit-log.md)
- [Admin billing management](billing-management.md)
