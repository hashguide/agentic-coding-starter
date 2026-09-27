---
kind: playbook
domain: operations
topics: [operations, admin]
authority: guidance
---

# Admin Panel Overview

Every SaaS product needs an internal panel for managing users, billing, feature access, and support operations. Admin capabilities built as an afterthought are a security risk — plan them from the start with access controls and audit logging wired in from day one.

## Core Admin Feature List

| Feature | Purpose |
|---|---|
| **User management** | Search users, view account details, suspend or delete, impersonate for support |
| **Billing management** | Look up Stripe customers, override plan access, issue refunds |
| **Feature flags** | Enable or disable features per user, per plan, or by percentage rollout |
| **Audit log** | Immutable record of every admin action with actor, target, timestamp, and changed values |

## Admin Authentication Requirements

Admin accounts require stronger authentication than regular user accounts. At minimum:

- MFA is **mandatory**, not optional
- Admin routes are protected at the middleware level, not only in the UI
- Admin sessions have shorter expiry windows than standard sessions

```typescript
// Middleware — applied to all /admin/* routes
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

  // Require recent MFA verification (within last 4 hours)
  const mfaAge = Date.now() - (user.mfaVerifiedAt?.getTime() ?? 0);
  if (mfaAge > 4 * 60 * 60 * 1000) {
    return res.status(403).json({ error: 'MFA_REQUIRED' });
  }

  next();
}

// Apply to all admin routes
app.use('/admin', requireAdmin);
```

## Security

Admin routes require MFA and must be protected at the middleware level independently of the UI. Every admin action is audit-logged.

**Why middleware, not UI-only:** UI checks can be bypassed by calling the API directly. The access check must be on the server, on every request.

**Why every action is audit-logged:** Without an audit trail, there is no way to investigate support issues, security incidents, or accidental data changes. The audit log is the admin panel's accountability layer.

## Related Documents

- [Admin user management](user-management.md)
- [Admin billing management](billing-management.md)
- [Admin feature flags](feature-flags.md)
- [Admin audit log](audit-log.md)
- [Auth MFA](../../access/authentication/mfa.md)
