---
kind: playbook
domain: experience
topics: [experience, responsive, system, theme]
authority: guidance
---

# System Theme Preferences

Users configure accessibility preferences at the operating system level. These preferences must be respected by default. Overriding them without the user's explicit in-app choice is an accessibility failure.

## `prefers-color-scheme`

Detects whether the user's OS is set to light or dark mode.

```css
/* Default: light mode styles in :root */
:root {
  --color-bg-base: #ffffff;
  --color-text-primary: #111827;
}

/* Automatically switch to dark when OS preference is dark */
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg-base: #0f172a;
    --color-text-primary: #f1f5f9;
  }
}
```

In JavaScript — detecting and responding to changes:

```typescript
const isDark = window.matchMedia('(prefers-color-scheme: dark)');

function applySystemTheme(mq: MediaQueryList | MediaQueryListEvent) {
  document.documentElement.dataset.theme = mq.matches ? 'dark' : 'light';
}

applySystemTheme(isDark);
isDark.addEventListener('change', applySystemTheme);
```

**Rule:** The system preference is the default. If the user has not set an explicit in-app theme override, the OS preference is followed. See [dark-mode.md](../visual/dark-mode.md) for user preference persistence.

## `prefers-reduced-motion`

Detects whether the user has requested reduced motion (often set by users with vestibular disorders, epilepsy, or motion sensitivity). When this preference is active, all non-essential animation should be removed or minimized.

```css
/* Disable animations globally when reduced motion is preferred */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

In JavaScript — for motion-library animations (Framer Motion, GSAP):

```typescript
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

// Framer Motion example
const transition = prefersReducedMotion
  ? { duration: 0 }
  : { duration: 0.3, ease: [0, 0, 0.2, 1] };
```

**What "essential" motion is:** Focus ring appearance, state changes that would be disorienting if they jumped without transition (e.g., a modal appearing without any enter animation). These can retain minimal (< 50ms) transitions even with reduced motion. Decorative animations, page transitions, micro-interactions, and loading animations should be fully disabled.

## `prefers-contrast`

Detects whether the user has requested increased contrast.

```css
@media (prefers-contrast: more) {
  :root {
    --color-border: #374151;            /* Stronger border */
    --color-text-secondary: #374151;    /* Stronger secondary text */
    --color-bg-muted: #e5e7eb;          /* More visible muted background */
  }
}
```

Less commonly implemented than the above two, but straightforward to support once the token system is in place.

## These Are Defaults, Not Locks

The system preferences are defaults. Users can override them explicitly in-app:
- A user can set their in-app theme to dark even if their OS is in light mode
- A user cannot currently override reduced-motion per-app at the OS level, so respect it unconditionally

The in-app override is stored in the user's profile (not just localStorage) so it persists across devices. When an explicit in-app preference exists, it takes precedence over the system preference.

## Related Documents

- [Dark mode implementation](../visual/dark-mode.md)
- [Motion principles](../visual/motion.md)
- [Accessibility compliance](../../compliance/accessibility-compliance.md)
