---
kind: playbook
domain: experience
topics: [experience, visual, color]
authority: guidance
---

# Color System

A structured color system makes UI consistent and accessible. Colors should be defined as semantic tokens with named purposes — not as raw hex values scattered across components.

## Palette Construction

**Step 1: Primary color.** Choose the brand's primary color — the action color used for buttons, links, and focus states. Generate a scale from 50 (near-white tint) to 900 (near-black shade).

**Step 2: Secondary color (optional).** A secondary accent used for non-primary interactive elements or callouts. Must not conflict with the semantic status colors.

**Step 3: Neutral scale.** A gray scale used for backgrounds, borders, and text. True gray (`#6B7280`) reads as neutral; cooler gray (with a slight blue cast) works well in modern dark-themed UIs.

**Step 4: Semantic status colors.** Four mandatory semantic colors with background variants:

| Purpose | Foreground | Background |
|---|---|---|
| **Success** | `#16a34a` (green-600) | `#f0fdf4` (green-50) |
| **Warning** | `#ca8a04` (yellow-600) | `#fefce8` (yellow-50) |
| **Error** | `#dc2626` (red-600) | `#fef2f2` (red-50) |
| **Info** | `#2563eb` (blue-600) | `#eff6ff` (blue-50) |

These values are specific enough to be immediately usable; adjust the specific hue to work with your brand primary while preserving the semantic meaning.

## Semantic Token Application

Map every use case to a semantic token. Never hardcode hex values in component styles:

```css
/* Background layers */
--color-bg-base: #ffffff;           /* Page background */
--color-bg-subtle: #f9fafb;         /* Sidebar, table header */
--color-bg-muted: #f3f4f6;          /* Input background, hover */
--color-bg-interactive: #e5e7eb;    /* Active/selected state */

/* Text */
--color-text-primary: #111827;      /* Headlines, body */
--color-text-secondary: #6b7280;    /* Labels, metadata */
--color-text-placeholder: #9ca3af;  /* Input placeholders */
--color-text-disabled: #d1d5db;     /* Disabled state */
--color-text-link: var(--color-primary); /* Links */

/* Borders */
--color-border: #e5e7eb;            /* Default border */
--color-border-strong: #d1d5db;     /* Emphasized border */
--color-border-focus: var(--color-primary); /* Focus ring */
```

## Accessibility Contrast Requirements

WCAG AA minimum contrast ratios:

| Text category | Minimum ratio |
|---|---|
| Body text (< 18px regular) | 4.5:1 |
| Large text (≥ 18px regular or ≥ 14px bold) | 3:1 |
| UI components and graphics | 3:1 |
| Decorative elements | None |

Test every color combination at the pixel level — the semantic color pair must meet contrast requirements not just in theory but against the background it actually appears on. Use the WebAIM Contrast Checker or the browser's DevTools accessibility panel.

## Color Is Never the Only Signal

Color alone must never be the only indicator of meaning. A user who is color-blind cannot distinguish between a red error state and a green success state if color is the only differentiator.

**Always pair color with:**
- An icon (✓ for success, ✕ for error, ⚠ for warning)
- Text (e.g., "Error: field required")
- A shape or position change

```tsx
// Wrong: color only
<div style={{ background: 'var(--color-error-bg)' }}>
  Your form contains errors.
</div>

// Right: color + icon + text
<div className="alert-error" role="alert">
  <AlertCircleIcon aria-hidden="true" />
  <span>Your form contains errors. See the highlighted fields below.</span>
</div>
```

## Dark Mode

In dark mode, the semantic token values swap — backgrounds become dark, text becomes light. See [dark-mode.md](dark-mode.md) for the implementation. The token naming stays the same; only the values change.

## Related Documents

- [Design tokens](../brand/tokens.md)
- [Dark mode](dark-mode.md)
- [Accessibility compliance](../../compliance/accessibility-compliance.md)
- [Frontend accessibility](../../frontend/accessibility.md)
