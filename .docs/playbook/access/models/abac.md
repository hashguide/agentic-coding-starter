---
kind: playbook
domain: access
topics: [access, models, abac]
authority: guidance
---

# Attribute-Based Access Control (ABAC)

ABAC evaluates permission based on attributes of the requesting user, the target resource, and the context of the request — not just the user's role. It is the right choice when permissions include conditional logic: "users with this role can edit this resource, but only if the resource belongs to their team and is not in a locked state."

## Definition

An ABAC policy is an expression that evaluates to "allow" or "deny" based on attributes:

**Subject attributes:** Who is making the request (user ID, role, team membership, subscription tier)

**Resource attributes:** What is being accessed (owner ID, team ID, status, visibility setting)

**Context attributes:** Conditions of the request (time of day, IP address, whether the user recently verified their password)

## Policy Example

```typescript
type PolicyContext = {
  user: {
    id: string;
    role: Role;
    teamId: string;
    subscriptionTier: 'free' | 'pro' | 'enterprise';
  };
  resource: {
    ownerId: string;
    teamId: string;
    status: 'draft' | 'published' | 'locked';
    visibility: 'private' | 'team' | 'public';
  };
};

// "User can edit document if:
//   - they own it (user.id == document.owner_id), OR
//   - they are in the same team (user.team_id == document.team_id)
//   AND the document is not locked"
function canEditDocument(ctx: PolicyContext): boolean {
  const isOwner = ctx.user.id === ctx.resource.ownerId;
  const isSameTeam = ctx.user.teamId === ctx.resource.teamId;
  const isNotLocked = ctx.resource.status !== 'locked';

  return (isOwner || isSameTeam) && isNotLocked;
}

// In a route handler
app.put('/api/documents/:id', requireAuth, asyncHandler(async (req, res) => {
  const document = await db.document.findUnique({ where: { id: req.params.id } });
  if (!document) throw new NotFoundError('DOCUMENT_NOT_FOUND');

  const user = await db.user.findUniqueOrThrow({
    where: { id: req.userId },
    select: { id: true, role: true, teamId: true, subscriptionTier: true },
  });

  const allowed = canEditDocument({
    user,
    resource: {
      ownerId: document.ownerId,
      teamId: document.teamId,
      status: document.status,
      visibility: document.visibility,
    },
  });

  if (!allowed) throw new ForbiddenError();

  const updated = await db.document.update({ where: { id: req.params.id }, data: req.body });
  res.json(updated);
}));
```

## Security

**ABAC policy evaluation is server-side only.** The server fetches resource attributes from the database and evaluates the policy. The client must never supply resource attributes to the policy evaluation — a client claiming `resource.teamId = attackerTeamId` to gain access is a trivial attack if the server trusts client-supplied attributes.

All attributes used in policy evaluation must be fetched from authoritative server-side sources (the database, the authenticated session) — never from the request body, query parameters, or any client-supplied data.

## Related Documents

- [RBAC access model](rbac.md)
- [RBAC vs ABAC](rbac-vs-abac.md)
- [Auth RBAC](../authentication/rbac.md)
