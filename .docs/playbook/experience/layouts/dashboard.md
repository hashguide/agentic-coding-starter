---
kind: playbook
domain: experience
topics: [experience, layouts, dashboard]
authority: guidance
---

# Dashboard Layout

The dashboard is the authenticated home — where returning users arrive after login. Its job is to surface the most important context and pending actions immediately, not to display a generic overview of everything.

## Navigation Shell Options

### Sidebar Navigation

A vertical navigation panel on the left side of the screen, persistent across all app views.

```
┌──────────┬──────────────────────────────────┐
│ Logo     │ [Page content]                   │
│ ──────── │                                  │
│ Nav item │                                  │
│ Nav item │                                  │
│ Nav item │                                  │
│          │                                  │
│ [User]   │                                  │
└──────────┴──────────────────────────────────┘
```

**Best for:** Products with 5+ primary navigation destinations, complex hierarchies, or where section switching is frequent. The sidebar makes all destinations visible at a glance.

**Trade-offs:** Takes horizontal space. On smaller screens, collapses to a hamburger menu or bottom nav.

### Top Navigation

A horizontal navigation bar at the top of the screen.

```
┌──────────────────────────────────────────────┐
│ Logo  Nav  Nav  Nav              [Bell][User] │
├──────────────────────────────────────────────┤
│ [Page content]                               │
└──────────────────────────────────────────────┘
```

**Best for:** Products with 3–5 primary destinations, simpler navigation trees, or where horizontal real estate is valuable (data tables, editors).

**Trade-offs:** Limited horizontal space for navigation items. Subnavigation requires dropdowns or a secondary nav row.

## Notification Bell Placement

The notification bell lives in the top-right of the navigation bar, regardless of whether the navigation shell uses a sidebar or top nav. It is always in the same position:

```
[Logo]  [Nav]  [Nav]  [Nav]  ——  [Bell] [User menu]
```

The bell shows an unread count badge when unread notifications exist; the badge disappears entirely at zero count. See [notifications.md](../interaction/notifications.md) for full notification system behavior.

## Widget / Card Grid for Metrics

Metric displays use a grid of equal-height cards, each representing a single metric or data point:

```
┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
│ MRR      │ │ Users    │ │ Churn    │ │ Trials   │
│ $12,400  │ │ 1,240    │ │ 2.1%     │ │ 38       │
│ +12% ↑   │ │ +8% ↑    │ │ -0.3% ↓  │ │ +5 today │
└──────────┘ └──────────┘ └──────────┘ └──────────┘
```

Rules:
- Limit to 4–6 metrics above the fold — more creates noise
- Show trend direction and magnitude, not just the raw value
- Color the trend indicator (positive/negative), but do not rely on color alone — include the arrow and the delta value

## What to Surface First

A returning user's dashboard should show **pending actions**, not generic aggregate statistics. Generic stats are the default because they are easy to build — not because they are useful.

**Pending action examples:**
- "2 invoices overdue — View invoices →"
- "3 users requested export — Review →"
- "Your trial ends in 7 days — Choose a plan →"
- "Team member Sarah has been waiting for an invite for 3 days"

Pending actions convert to completed tasks. Statistics just inform.

## Related Documents

- [Notifications](../interaction/notifications.md)
- [Data lists](data-lists.md)
- [Responsive layout patterns](../responsive/layout-patterns.md)
- [Returning user UX flow](../journeys/returning-user.md)
