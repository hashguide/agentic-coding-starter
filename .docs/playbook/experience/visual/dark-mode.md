---
kind: playbook
domain: experience
topics: [experience, visual, dark, mode]
authority: guidance
---

# Dark Mode

Dark mode is an expected feature in modern products. Users who prefer it are vocal and will notice its absence. Implementing dark mode correctly requires semantic color tokens and a clear preference persistence strategy — not a CSS `filter: invert()` hack.

## Semantic Tokens Are the Foundation

Dark mode works correctly only when every UI color is referenced by a semantic token, never by a raw hex value. The implementation is: swap token values in dark mode. All components automatically adapt.

```css
:root {
  --color-bg-base: #ffffff;
  --color-bg-subtle: #f9fafb;
  --color-text-primary: #111827;
  --color-text-secondary: #6b7280;
  --color-border: #e5e7eb;
}

[data-theme='dark'] {
  --color-bg-base: #0f172a;
  --color-bg-subtle: #1e293b;
  --color-text-primary: #f1f5f9;
  --color-text-secondary: #94a3b8;
  --color-border: #334155;
}
```

Every component that uses `var(--color-bg-base)` adapts to dark mode automatically. A component that uses `#ffffff` hardcoded does not.

## System Theme Detection

Respect the user's OS preference by default. The `prefers-color-scheme` media query provides this:

```css
@media (prefers-color-scheme: dark) {
  :root {
    /* Same values as [data-theme='dark'] above */
    --color-bg-base: #0f172a;
    /* ... */
  }
}
```

In JavaScript/React:

```typescript
function useSystemTheme(): 'light' | 'dark' {
  const [theme, setTheme] = React.useState<'light' | 'dark'>(
    window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light'
  );

  React.useEffect(() => {
    const mq = window.matchMedia('(prefers-color-scheme: dark)');
    const handler = (e: MediaQueryListEvent) => setTheme(e.matches ? 'dark' : 'light');
    mq.addEventListener('change', handler);
    return () => mq.removeEventListener('change', handler);
  }, []);

  return theme;
}
```

## Persisting User Theme Preference

The user's explicit theme choice (if they override the system default) must be persisted to their profile in the database — not just to `localStorage`.

**Why:** `localStorage` is device-specific. A user who sets dark mode on their laptop and then opens the app on their phone will see light mode. Saving to the user profile syncs the preference across devices.

```typescript
// User profile schema
model User {
  themePreference  String?  // 'light' | 'dark' | 'system' | null (= 'system')
}

// Theme resolution logic
function resolveTheme(userPreference: string | null, systemTheme: 'light' | 'dark') {
  if (!userPreference || userPreference === 'system') return systemTheme;
  return userPreference as 'light' | 'dark';
}
```

On page load, apply the theme immediately from a cookie or server-rendered attribute to avoid a flash of wrong theme before JavaScript executes:

```tsx
// In your root layout (e.g., Next.js app/layout.tsx)
export default async function RootLayout({ children }) {
  const session = await getSession();
  const theme = session?.user?.themePreference ?? 'system';

  return (
    <html data-theme={theme} lang="en">
      <body>{children}</body>
    </html>
  );
}
```

## Image Handling in Dark Mode

Do not invert images with `filter: invert(1)` in dark mode. This turns photographs negative and makes logos unrecognizable.

Correct approaches:
- **Logos:** Provide dark-mode variants and switch via `<picture>` and `prefers-color-scheme` media query in CSS
- **Photographs:** No change needed — photographs look fine on dark backgrounds
- **Illustrations / icons:** Provide dark variants or design them to work on both backgrounds

```html
<picture>
  <source srcset="logo-dark.svg" media="(prefers-color-scheme: dark)" />
  <img src="logo-light.svg" alt="Product logo" />
</picture>
```

## Related Documents

- [Design tokens](../brand/tokens.md)
- [System theme detection](../responsive/system-theme.md)
- [Design system color](color.md)
