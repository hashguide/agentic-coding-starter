---
kind: playbook
domain: experience
topics: [experience, responsive, layout, patterns]
authority: guidance
---

# Responsive Layout Patterns

Responsive layout patterns are the recurring transformations that occur as screens shrink. Having a named pattern for each transformation keeps implementation consistent.

## Multi-Column to Single-Column Stack

The most common transformation. A multi-column layout on desktop collapses to a single-column stack on mobile.

```css
.feature-grid {
  display: grid;
  grid-template-columns: 1fr; /* Mobile: single column */
  gap: var(--space-6);
}

@media (min-width: 768px) {
  .feature-grid {
    grid-template-columns: 1fr 1fr; /* Tablet: 2 columns */
  }
}

@media (min-width: 1024px) {
  .feature-grid {
    grid-template-columns: 1fr 1fr 1fr; /* Desktop: 3 columns */
  }
}
```

**When:** Feature sections, card grids, pricing tiers, testimonials.

**Ordering note:** In single-column, visual order matches DOM order. Verify the stacking order makes logical sense — a card that appears second in a two-column desktop layout may need to move to first in the mobile stack.

## Sidebar to Bottom Nav or Hamburger Menu

A persistent sidebar navigation on desktop collapses to either:
- **Bottom navigation bar:** 4–5 primary items pinned to the bottom of the viewport — works best for apps with a small, flat nav structure
- **Hamburger menu:** A toggle that opens a slide-over drawer — works for complex nav hierarchies

```css
.sidebar {
  display: none;
  width: 240px;
  position: sticky;
  top: 0;
  height: 100vh;
}

.bottom-nav {
  display: flex;
  position: fixed;
  bottom: 0;
  width: 100%;
}

@media (min-width: 1024px) {
  .sidebar { display: block; }
  .bottom-nav { display: none; }
}
```

**When to use bottom nav:** Consumer apps, products used primarily on mobile, navigation with ≤ 5 primary destinations.

**When to use hamburger:** Complex nav structures with nested items or more than 5 top-level items, B2B dashboards where desktop is primary and mobile is supplementary.

## Data Table to Card List

Data tables with many columns become unreadable on small screens. The responsive transformation converts each table row into a card displaying the same data in a vertical layout.

```tsx
function ResponsiveDataTable({ data, columns }: TableProps) {
  return (
    <>
      {/* Table — hidden on mobile */}
      <table className="hidden md:table">
        <thead>
          <tr>{columns.map((col) => <th key={col.key}>{col.label}</th>)}</tr>
        </thead>
        <tbody>
          {data.map((row) => (
            <tr key={row.id}>
              {columns.map((col) => <td key={col.key}>{row[col.key]}</td>)}
            </tr>
          ))}
        </tbody>
      </table>

      {/* Card list — visible on mobile only */}
      <ul className="md:hidden">
        {data.map((row) => (
          <li key={row.id} className="data-card">
            {columns.map((col) => (
              <div key={col.key} className="data-card-field">
                <span className="field-label">{col.label}</span>
                <span className="field-value">{row[col.key]}</span>
              </div>
            ))}
          </li>
        ))}
      </ul>
    </>
  );
}
```

**When:** Invoice lists, user tables, order histories — any tabular data with more than 3 columns.

**Alternative:** On mobile, show only the 2–3 most important columns rather than switching to cards. This works when the data is scannable enough in truncated form.

## Related Documents

- [Responsive overview](overview.md)
- [Breakpoints](breakpoints.md)
- [Dashboard layout](../layouts/dashboard.md)
- [Data lists layout](../layouts/data-lists.md)
