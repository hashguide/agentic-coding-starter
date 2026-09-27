---
kind: playbook
domain: experience
topics: [experience, layouts, data, lists]
authority: guidance
---

# Data Lists

Data lists display collections of items — invoices, users, projects, orders. The right display format and pagination strategy depends on the data type, the number of items, and how users interact with the list.

## Table vs. Card Grid vs. Simple List

| Format | Best for | Avoid when |
|---|---|---|
| **Table** | Structured data with multiple comparable attributes (invoices, users, orders) where scanning columns matters | Items have varied or rich content (articles, projects with images) |
| **Card grid** | Items with visual hierarchy, preview images, or varied content (projects, templates, content pieces) | The data is tabular and column comparison is important |
| **Simple list** | Linear items where the primary action is to select or navigate to one item (notification list, search results, recent items) | You need column-level sorting or filtering |

**Decision shortcut:** If users will sort by a column or compare values across rows, use a table. If users browse to find and open an item, use cards or a list.

## Pagination Options

| Pattern | Description | Best for |
|---|---|---|
| **Numbered pages** | Page 1, 2, 3... with previous/next controls | Large datasets where users need to jump to a specific page or return to a known position |
| **Infinite scroll** | Automatically loads the next page as the user scrolls to the bottom | Social feeds, content discovery, exploratory browsing where exact position doesn't matter |
| **Load more button** | Manual trigger to append next page | Analytics dashboards, historical records — gives user control, avoids position disorientation |

**Trade-offs:**

Infinite scroll is the right choice for casual browsing. It breaks for lists where users need to navigate back to a specific item (the scroll position is lost on navigation). Load more + anchor links or numbered pagination handle this better.

Numbered pages are the right choice for large tabular datasets (thousands of rows). Users can say "the invoice I'm looking for is on page 4."

## Sorting and Filtering

Sorting is controlled by column headers in tables (click header → sort ascending → click again → sort descending → unsorted). Show the active sort direction with an arrow icon on the active column.

Filters live above the list in a filter bar. Active filters are shown as chips/tags that can be individually dismissed.

## Batch Actions

For lists where users frequently act on multiple items simultaneously, support batch selection:
- Checkbox on each row (or card)
- Select all checkbox in the table header
- Action toolbar that appears when any items are selected ("Delete selected", "Export selected")

## Empty States

Every list, table, and card grid needs an empty state. See [empty-states.md](../interaction/empty-states.md).

Distinguish:
- **True empty:** No items exist — show a create/add CTA
- **Filtered empty:** Filters returned no results — show "clear filters" CTA

## Related Documents

- [Component states (table states)](../interaction/component-states.md)
- [Empty states](../interaction/empty-states.md)
- [Responsive layout patterns (table to card transformation)](../responsive/layout-patterns.md)
