---
kind: playbook
domain: experience
topics: [experience, visual, motion]
authority: guidance
---

# Motion

Motion in UI communicates state changes, establishes hierarchy, and guides attention. Motion that decorates without communicating is expensive (performance, attention, accessibility) and adds nothing. The principle: motion should clarify, not decorate.

## Motion Duration Scale

```css
:root {
  --duration-fast: 100ms;       /* Hover state transitions, toggles */
  --duration-base: 200ms;       /* Standard UI transitions — buttons, inputs, dropdown */
  --duration-slow: 300ms;       /* Larger elements entering/exiting — modals, drawers */
  --duration-deliberate: 500ms; /* Emphasis animations — success states, empty state illustrations */
}
```

**Use the shortest duration that still reads as intentional.** A button hover at 100ms is fast enough to feel responsive; at 300ms it feels sluggish. A modal entering at 100ms feels jarring; at 300ms it feels controlled.

## Easing Curves

```css
:root {
  --ease-default: cubic-bezier(0.4, 0, 0.2, 1);     /* Smooth in-out — standard transitions */
  --ease-in: cubic-bezier(0.4, 0, 1, 1);             /* Accelerating — elements exiting */
  --ease-out: cubic-bezier(0, 0, 0.2, 1);            /* Decelerating — elements entering */
  --ease-spring: cubic-bezier(0.34, 1.56, 0.64, 1);  /* Slight overshoot — playful, use sparingly */
}
```

**Rule:**
- Entering elements use `ease-out` — they decelerate as they arrive, giving a sense of mass settling
- Exiting elements use `ease-in` — they accelerate as they leave, getting out of the way quickly
- Standard transitions use `ease-default`

## Motion Catalog by Use Case

| Moment | Duration | Easing | Effect |
|---|---|---|---|
| Button hover | `fast` (100ms) | `ease-default` | Background color transition |
| Button press | `fast` (100ms) | `ease-in` | `scale(0.98)` |
| Input focus | `fast` (100ms) | `ease-default` | Border color transition |
| Toggle switch | `base` (200ms) | `ease-default` | Thumb translate + track color |
| Dropdown open | `base` (200ms) | `ease-out` | Fade in + slide down 4px |
| Dropdown close | `fast` (100ms) | `ease-in` | Fade out |
| Modal open | `slow` (300ms) | `ease-out` | Fade in backdrop + scale(0.95→1) + fade in panel |
| Modal close | `base` (200ms) | `ease-in` | Fade out |
| Toast enter | `base` (200ms) | `ease-out` | Slide in from edge |
| Toast exit | `base` (200ms) | `ease-in` | Fade out |
| Skeleton → content | `base` (200ms) | `ease-default` | Fade in content |
| Success checkmark | `deliberate` (500ms) | `ease-spring` | Draw-on animation |

## Reduced Motion

Respect the user's `prefers-reduced-motion` preference. Users who have this set (often due to vestibular disorders) find motion nauseating. Remove non-essential animation:

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

In JavaScript:

```typescript
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

const transition = prefersReducedMotion
  ? { duration: 0 }
  : { duration: 0.3, ease: [0, 0, 0.2, 1] };
```

## What Not to Animate

- Scrolling containers (parallax effects cause motion sickness)
- Large-scale background motion
- Continuous looping animations in the viewport (distract attention from content)
- Micro-interactions on every hover if the UI has high element density

## Related Documents

- [Design tokens (motion tokens)](../brand/tokens.md)
- [Micro-interactions](../interaction/micro-interactions.md)
- [System theme (prefers-reduced-motion)](../responsive/system-theme.md)
- [Accessibility compliance](../../compliance/accessibility-compliance.md)
