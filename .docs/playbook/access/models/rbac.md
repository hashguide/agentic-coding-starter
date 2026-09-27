---
kind: playbook
domain: access
topics: [access, models, rbac]
authority: guidance
---

# RBAC as an Access Model

RBAC (Role-Based Access Control) assigns permissions to roles rather than to individual users. Users are assigned roles; roles determine what they can do. It is the right access model when permission logic is fully described by "what role does this user have" — without conditional "but only if" logic.

## Role Hierarchy Concept

Roles are typically hierarchical: each role includes all permissions of the roles below it.

```
Owner
  └── Admin
        └── Member
              └── Viewer
```

- **Viewer:** Read-only access to shared resources
- **Member:** Read/write on their own resources; read on shared resources
- **Admin:** Member permissions + user management, team settings
- **Owner:** Admin permissions + billing, account deletion, full control

This hierarchy simplifies assignment: promoting a Member to Admin automatically grants all Member permissions. Defining permissions per role rather than per individual avoids the combinatorial explosion of user-level permission management.

## Implementation Example

```typescript
// Permission definitions
const PERMISSIONS = {
  'invoices:read': ['viewer', 'member', 'admin', 'owner'],
  'invoices:write': ['member', 'admin', 'owner'],
  'invoices:delete': ['admin', 'owner'],
  'members:invite': ['admin', 'owner'],
  'billing:manage': ['owner'],
  'account:delete': ['owner'],
} as const;

type Permission = keyof typeof PERMISSIONS;
type Role = 'viewer' | 'member' | 'admin' | 'owner';

// Check permission in middleware
function requirePermission(permission: Permission) {
  return async (req: Request, res: Response, next: NextFunction): Promise<void> => {
    if (!req.userId) {
      res.status(401).json({ code: 'AUTH_REQUIRED' });
      return;
    }

    const member = await db.organizationMember.findFirst({
      where: { userId: req.userId, organizationId: req.organizationId },
      select: { role: true },
    });

    const allowedRoles = PERMISSIONS[permission] as readonly string[];
    if (!member || !allowedRoles.includes(member.role)) {
      res.status(403).json({ code: 'FORBIDDEN' });
      return;
    }

    next();
  };
}

// Route using permission check
app.delete(
  '/api/invoices/:id',
  requireAuth,
  requirePermission('invoices:delete'),
  asyncHandler(async (req, res) => {
    await invoiceService.softDelete(req.params.id, req.userId!);
    res.json({ success: true });
  }),
);
```

## Security

**Enforcement belongs at the DB and API layer, not just the UI.** Hiding a button or tab in the React UI is a UX decision. The API route must enforce the permission independently, because any user can make a direct HTTP request to the API without using the UI.

The permission check in the middleware above queries the database for the user's current role on every request. The role is not cached in the session (a user promoted from member to admin would still have member permissions until their session expires) unless you implement cache invalidation on role change.

## Related Documents

- [Auth RBAC](../authentication/rbac.md)
- [ABAC](abac.md)
- [RBAC vs ABAC](rbac-vs-abac.md)
- [Multi-tenant](multi-tenant.md)
