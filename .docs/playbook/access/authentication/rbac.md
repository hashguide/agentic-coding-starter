---
kind: playbook
domain: access
topics: [access, authentication, rbac]
authority: guidance
---

# Role-Based Access Control (RBAC)

RBAC controls what authenticated users are allowed to do. A user is authenticated (we know who they are) and authorized (we determine what they can access) through separate mechanisms. RBAC is the authorization mechanism: roles define permissions, and every request is checked against the user's role before the operation proceeds.

## Roles Schema

```typescript
// In schema.prisma
enum Role {
  owner   // Full access; billing; can delete account
  admin   // Manage members; access all features
  member  // Standard access
  viewer  // Read-only access
}

model User {
  id        String   @id @default(uuid())
  email     String   @unique
  role      Role     @default(member)
  // ...
}
```

For multi-tenant products with organization-level roles (a user may be an admin in one org and a member in another):

```typescript
model OrganizationMember {
  id             String   @id @default(uuid())
  userId         String
  organizationId String
  role           Role     @default(member)

  user           User         @relation(fields: [userId], references: [id])
  organization   Organization @relation(fields: [organizationId], references: [id])

  @@unique([userId, organizationId])
}
```

## Middleware Permission Check

Role enforcement happens in middleware, before route handlers execute. This ensures authorization is applied consistently without relying on every handler to remember to check it.

```typescript
function requireRole(...allowedRoles: Role[]) {
  return async (req: Request, res: Response, next: NextFunction): Promise<void> => {
    if (!req.userId) {
      res.status(401).json({ code: 'AUTH_REQUIRED' });
      return;
    }

    const user = await db.user.findUnique({
      where: { id: req.userId },
      select: { role: true },
    });

    if (!user || !allowedRoles.includes(user.role)) {
      res.status(403).json({ code: 'FORBIDDEN' });
      return;
    }

    next();
  };
}

// Usage — admin-only route
app.delete(
  '/api/users/:id',
  requireAuth,
  requireRole('owner', 'admin'),
  asyncHandler(async (req, res) => {
    await userService.delete(req.params.id);
    res.json({ success: true });
  }),
);
```

## Security

**Role verification happens server-side on every request.** A client claiming to be an admin is an untrusted assertion until the database confirms it. Never trust role claims from:
- JWT payload (a modified JWT could claim any role)
- Request body or query parameters
- HTTP headers set by the client
- LocalStorage or cookies that are readable by JavaScript

The only trusted source of a user's role is a database lookup using the session/token that the server controls.

**Hiding a UI element is not access control.** If the admin button is hidden via CSS or conditional rendering, but the underlying API route is not protected, any user can call the API directly. UI hiding is UX; API protection is security. Both are required, and API protection is the one that cannot be bypassed.

## Related Documents

- [Auth options](options.md)
- [Auth session management](session-management.md)
- [Access models RBAC](../models/rbac.md)
- [Access models overview](../models/overview.md)
