---
kind: playbook
domain: experience
topics: [experience, responsive, breakpoints]
authority: guidance
---

# Breakpoints

A breakpoint system defines the screen-width thresholds at which layouts change. A consistent, named system is preferable to arbitrary pixel values scattered throughout the codebase.

## Standard Breakpoint Scale

This scale matches Tailwind CSS's default breakpoints and is a reasonable starting point for most products:

| Name | Min-width | Typical context |
|---|---|---|
| `sm` | 640px | Large phones (landscape), small tablets |
| `md` | 768px | Tablets (portrait) |
| `lg` | 1024px | Small desktops, large tablets (landscape) |
| `xl` | 1280px | Standard desktop monitors |
| `2xl` | 1536px | Large / wide monitors |

```css
/* In CSS — mobile-first (min-width) */
.grid { grid-template-columns: 1fr; }

@media (min-width: 768px)  { .grid { grid-template-columns: 1fr 1fr; } }
@media (min-width: 1024px) { .grid { grid-template-columns: 1fr 1fr 1fr; } }
```

In Tailwind:
```html
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3">
```

## Container Queries

Container queries adapt a component based on the size of its containing element, not the viewport. This solves the problem where the same component appears in both a narrow sidebar and a wide main panel — a viewport breakpoint can't distinguish these contexts.

```css
.card-container {
  container-type: inline-size;
  container-name: card;
}

.card-body {
  flex-direction: column;
}

@container card (min-width: 400px) {
  .card-body {
    flex-direction: row;
  }
}
```

**When to use container queries:** When a component's layout should respond to its container width rather than the viewport — especially reusable components placed in multiple contexts.

**Browser support:** Container queries are supported in all modern browsers (Chrome 105+, Firefox 110+, Safari 16+). Check [caniuse.com](https://caniuse.com/css-container-queries) if you need to support older browsers.

## Fluid vs. Fixed Breakpoints

**Fixed breakpoint jumps:** Layout changes abruptly at defined thresholds. Appropriate for structural changes (sidebar appears/disappears, column count changes).

**Fluid sizing:** Values scale continuously between min and max values. No abrupt jump — the layout breathes with the viewport. Use CSS `clamp()`:

```css
/* Fluid type: scales from 16px (mobile) to 20px (desktop) */
.lead {
  font-size: clamp(1rem, 1.5vw + 0.5rem, 1.25rem);
}

/* Fluid spacing: scales from 16px to 48px */
.section-padding {
  padding-block: clamp(1rem, 5vw, 3rem);
}
```

**When fluid is preferable:**
- Typography sizes — continuous scaling reads more naturally than discrete jumps
- Section padding — breathing room that scales with available space
- Hero elements — text and images that fill a header fluidly

**When fixed breakpoints are preferable:**
- Column layout changes (1→2→3 columns)
- Navigation transformations (sidebar → hamburger menu)
- Component visibility toggling

## Related Documents

- [Responsive overview](overview.md)
- [Layout patterns](layout-patterns.md)
- [Design tokens (spacing scale)](../brand/tokens.md)
