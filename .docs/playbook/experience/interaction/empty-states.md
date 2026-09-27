---
kind: playbook
domain: experience
topics: [experience, interaction, empty, states]
authority: guidance
---

# Empty States

An empty state occurs whenever a container has no content to display. No UI container should ever be left blank — a blank space looks broken and gives the user no path forward. Every list, table, panel, or section that can be empty needs a designed empty state.

## The Rule

No UI container is ever rendered blank. If a query returns zero results, a list has never been populated, or all items have been deleted, show an empty state — not nothing.

## Empty State Template

Every proper empty state has three parts:

```
[Illustration or icon]
[One-sentence explanation]
[Direct CTA button]
```

- **Illustration or icon:** A thematic icon or simple illustration that matches the content type (folder icon for projects, invoice icon for billing, bell icon for notifications). Keep illustrations simple — they are wayfinding elements, not hero artwork.
- **Explanation:** One sentence that tells the user why this space is empty and sets context. Write it from the user's perspective.
- **CTA:** A direct action button that moves the user toward populating this space. Make it specific to the content type — not a generic "Get started."

```tsx
function EmptyState({ icon: Icon, title, description, action }: EmptyStateProps) {
  return (
    <div className="empty-state" role="status">
      <Icon className="empty-state-icon" aria-hidden="true" />
      <p className="empty-state-title">{title}</p>
      {description && <p className="empty-state-description">{description}</p>}
      {action && (
        <a href={action.href} className="btn-primary">
          {action.label}
        </a>
      )}
    </div>
  );
}
```

## Example Empty States

**No projects yet:**
```
[FolderIcon]
"You don't have any projects yet."
[Create your first project →]
```

**No search results:**
```
[SearchIcon]
"No projects match your search."
[Clear search]
```

**No notifications:**
```
[BellIcon]
"You're all caught up — no new notifications."
[no CTA needed]
```

**No invoices:**
```
[ReceiptIcon]
"No invoices yet. Invoices you send will appear here."
[Create invoice →]
```

**No team members:**
```
[UsersIcon]
"It's just you right now. Invite your team to collaborate."
[Invite team member →]
```

## Filtered vs. True Empty

Distinguish between a truly empty state and a filtered-no-results state:

| Type | Explanation | CTA |
|---|---|---|
| True empty | First-time: no data exists | Create first item |
| Filtered empty | A filter or search returned no results | Clear filters |

These require different copy and different CTAs. A first-time empty state with a "Clear search" button is confusing. A filtered empty with a "Create your first project" button when there are actually 50 projects is wrong.

## Related Documents

- [Component states](component-states.md)
- [Notifications empty state](notifications.md)
- [Micro-content](micro-content.md)
