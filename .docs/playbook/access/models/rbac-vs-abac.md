---
kind: playbook
domain: access
topics: [access, models, rbac, vs, abac]
authority: guidance
---

# RBAC vs. ABAC

The choice between RBAC and ABAC is a function of how complex the permission logic is. RBAC is simpler to implement and understand; ABAC is more expressive. Most products start with RBAC and extend with ABAC where conditions require it.

## The Decision Rule

**Use RBAC if:** Permission logic is fully described by "what role does this user have?" Every question about whether a user can perform an action is answered by their role alone.

**Use ABAC (or RBAC + ABAC) if:** "But only if" conditional logic appears on top of roles. The resource's attributes (owner, status, team membership) determine permission in addition to the user's role.

## Decision Examples

| Permission question | Answer type | Model |
|---|---|---|
| Can this user invite team members? | Only admins can → role-only | RBAC |
| Can this user view this invoice? | Only if they own it or are an admin → role + resource attribute | RBAC + ABAC |
| Can this user edit this document? | Only if in the same team AND the document is not locked | ABAC |
| Can this user access the billing page? | Only if they have the 'owner' role → role-only | RBAC |
| Can this user download this export? | Only if the export belongs to their account → resource attribute | ABAC |

## Worked Examples

### Pure RBAC

```typescript
// "Can the user manage billing?"
// Answer: yes if they are an owner; no otherwise
function canManageBilling(userRole: Role): boolean {
  return userRole === 'owner';
}
```

No resource attributes needed — the role fully determines the answer.

### Pure ABAC

```typescript
// "Can the user access this download link?"
// Answer: yes if the download belongs to their order
function canAccessDownload(userId: string, download: Download): boolean {
  return download.userId === userId;
}
```

No role needed — the resource attribute fully determines the answer.

### RBAC + ABAC Hybrid

```typescript
// "Can the user edit this invoice?"
// Answer: admins can edit any invoice; members can only edit their own
function canEditInvoice(user: User, invoice: Invoice): boolean {
  if (user.role === 'admin' || user.role === 'owner') return true;
  if (user.role === 'member') return invoice.userId === user.id;
  return false; // viewers cannot edit
}
```

Role and resource attribute together determine the answer. This is the most common pattern in team SaaS products.

## When to Combine

Start with RBAC. Add ABAC conditions when:
1. You need to distinguish "user can edit their own X" from "admin can edit any X"
2. Resource status affects permission (locked, archived, published)
3. Visibility/privacy settings determine access

Extract ABAC conditions into named policy functions (as shown above) to keep the logic testable and readable.

## Related Documents

- [RBAC access model](rbac.md)
- [ABAC access model](abac.md)
- [Access model selection](access-model-selection.md)
