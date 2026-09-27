---
kind: playbook
domain: experience
topics: [experience, responsive]
authority: guidance
---

# Responsive Design Philosophy

Responsive design means the layout and content adapt to the size and capability of the device displaying it. The two dominant approaches — mobile-first and desktop-first — produce different CSS architectures and carry different implicit assumptions about who is using the product.

## Mobile-First CSS

Mobile-first writes base styles for small screens and uses `min-width` media queries to progressively enhance for larger screens:

```css
/* Base styles — applied to all screens (including mobile) */
.sidebar {
  display: none;
}

/* Enhancement for larger screens */
@media (min-width: 1024px) {
  .sidebar {
    display: block;
    width: 240px;
  }
}
```

**Philosophy:** Start with the simplest, most constrained version of the UI. Add complexity only where there is space to accommodate it. This is progressive enhancement.

**Practical benefit:** Mobile styles are typically simpler — single column, larger touch targets, fewer columns. Starting there means you never need to "undo" complex styles for small screens.

## Desktop-First CSS

Desktop-first writes base styles for large screens and uses `max-width` media queries to adapt for smaller screens:

```css
/* Base styles — for large screens */
.sidebar {
  display: block;
  width: 240px;
}

/* Override for small screens */
@media (max-width: 1023px) {
  .sidebar {
    display: none;
  }
}
```

**Philosophy:** Design the full-featured version first, then remove or collapse features for constrained screens.

**When it fits:** Products with complex desktop workflows (dashboards, editors, IDEs) where the desktop experience is primary and mobile is a simplified companion. The full product simply doesn't fit on mobile.

## Recommendation

**Mobile-first for most SaaS products.** Even B2B dashboards have mobile users — checking status on the way to a meeting, responding to a notification on a phone. Starting mobile-first ensures the most-constrained experience is explicit and intentional, not an afterthought.

**Desktop-first if:** the product's core workflow is irreducibly desktop-only (e.g., a code editor, a data visualization tool with complex interactions) and mobile is explicitly out of scope.

**Either way:** Define breakpoints before writing layout code. An ad-hoc collection of arbitrary `max-width: 768px` values scattered across a codebase is not a responsive design system. See [breakpoints.md](breakpoints.md) for a consistent scale.

## Related Documents

- [Breakpoints](breakpoints.md)
- [Layout patterns](layout-patterns.md)
- [System theme](system-theme.md)
