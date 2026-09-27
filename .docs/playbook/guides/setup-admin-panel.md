---
kind: playbook
domain: guides
topics: [guides, setup, admin, panel]
authority: guidance
---

# How to Setup an Admin Panel

**What you'll have at the end:** A secured, functioning admin panel with user search, Stripe billing lookup, audit logging, and verified access control.

---

## Step 1 — Add the Admin Role to Your User Schema

```typescript
// prisma/schema.prisma
model User {
  // ...existing fields...
  role  Role  @default(USER)
}

enum Role {
  USER
  ADMIN
}
```

```bash
npx prisma migrate dev --name add-admin-role
```

Assign yourself admin access:

```bash
npx prisma studio
# Or via psql/migration:
# UPDATE "User" SET role = 'ADMIN' WHERE email = 'your@email.com';
```

---

## Step 2 — Implement Admin Route Protection Middleware

```typescript
// middleware/requireAdmin.ts
export async function requireAdmin(req: Request, res: Response, next: NextFunction) {
  if (!req.session.userId) {
    return res.status(401).json({ error: 'Unauthenticated' });
  }

  const user = await db.user.findUnique({
    where: { id: req.session.userId },
    select: { role: true, mfaVerifiedAt: true },
  });

  if (user?.role !== 'ADMIN') {
    return res.status(403).json({ error: 'Forbidden' });
  }

  // Require MFA verification within the last 4 hours
  const mfaAge = Date.now() - (user.mfaVerifiedAt?.getTime() ?? 0);
  if (mfaAge > 4 * 60 * 60 * 1000) {
    return res.status(403).json({ error: 'MFA_REQUIRED' });
  }

  next();
}

// Apply to all /admin routes
app.use('/admin', requireAdmin);
```

---

## Step 3 — Build the User List and Search View

```typescript
// GET /admin/users?q=email
app.get('/admin/users', requireAdmin, async (req, res) => {
  const { q } = req.query as { q?: string };

  const users = await db.user.findMany({
    where: q ? { OR: [{ email: { contains: q } }, { name: { contains: q } }] } : {},
    select: { id: true, email: true, name: true, role: true, subscriptionStatus: true, createdAt: true },
    take: 25,
    orderBy: { createdAt: 'desc' },
  });

  res.json(users);
});
```

---

## Step 4 — Wire Up Stripe Customer Lookup

```typescript
// GET /admin/users/:userId/billing
app.get('/admin/users/:userId/billing', requireAdmin, async (req, res) => {
  const user = await db.user.findUnique({
    where: { id: req.params.userId },
    select: { stripeCustomerId: true },
  });

  if (!user?.stripeCustomerId) {
    return res.json({ subscriptions: [], invoices: [] });
  }

  const [subscriptions, invoices] = await Promise.all([
    stripe.subscriptions.list({ customer: user.stripeCustomerId, limit: 5 }),
    stripe.invoices.list({ customer: user.stripeCustomerId, limit: 10 }),
  ]);

  res.json({ subscriptions: subscriptions.data, invoices: invoices.data });
});
```

---

## Step 5 — Implement the Audit Log

```typescript
// lib/auditLog.ts
export async function writeAuditLog(entry: {
  actorId: string;
  action: string;
  targetId?: string;
  metadata?: Record<string, unknown>;
}) {
  await db.auditLog.create({ data: entry });
}
```

Call `writeAuditLog` in every admin action handler. Example:

```typescript
app.post('/admin/users/:userId/suspend', requireAdmin, async (req, res) => {
  await db.user.update({ where: { id: req.params.userId }, data: { suspendedAt: new Date() } });
  await writeAuditLog({ actorId: req.session.userId!, action: 'user.suspend', targetId: req.params.userId });
  res.json({ success: true });
});
```

See [admin/audit-log.md](../operations/admin/audit-log.md) for the full schema and append-only enforcement.

---

## Step 6 — Verify Access Control

```bash
# Test 1: Non-admin user cannot access admin routes
curl -H "Cookie: session=<non-admin-session>" http://localhost:3000/admin/users
# Should return: 403 Forbidden

# Test 2: Unauthenticated request cannot access admin routes
curl http://localhost:3000/admin/users
# Should return: 401 Unauthenticated

# Test 3: Admin user can access admin routes
curl -H "Cookie: session=<admin-session>" http://localhost:3000/admin/users
# Should return: 200 with user list
```

---

## Verify

- [ ] Non-admin users receive 403 on all `/admin/*` routes
- [ ] Admin access check is in middleware, not only in the UI
- [ ] Audit log record is created for every admin action
- [ ] Stripe billing lookup returns subscription and invoice data
- [ ] MFA gate blocks admin access when MFA is expired

---

## Related Documents

- [Admin overview](../operations/admin/overview.md)
- [Admin user management](../operations/admin/user-management.md)
- [Admin billing management](../operations/admin/billing-management.md)
- [Admin audit log](../operations/admin/audit-log.md)
