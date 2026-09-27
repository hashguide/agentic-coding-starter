---
kind: playbook
domain: guides
topics: [guides, setup, cookie, consent]
authority: guidance
---

# How to Setup Cookie Consent

**What you'll have at the end:** A compliant cookie consent flow that blocks non-essential scripts until consent is given.

---

## Step 1 — Audit All Cookies and Scripts

Before writing any code, list every third-party script and cookie your application sets:

```
Strictly necessary:
- Session cookie (auth)
- CSRF token cookie

Analytics (requires consent):
- PostHog / Mixpanel / Google Analytics

Marketing (requires consent):
- Facebook Pixel, Google Ads

Preferences (requires consent in strict interpretation):
- Dark mode preference stored in cookie (safe to store in localStorage instead)
```

Scripts in the "analytics" or "marketing" categories must not load until the user consents.

---

## Step 2 — Choose a Consent Banner Solution

**Option A: Build your own** — straightforward for small projects. See [compliance/cookie-consent.md](../compliance/cookie-consent.md) for the implementation pattern.

**Option B: Cookiebot or Usercentrics** — paid, handles geo-detection, pre-approved consent strings, and legal updates automatically. Worth it for products with EU users and complex cookie stacks.

**Option C: Osano** — offers a free tier, handles IAB TCF consent framework.

For most solo builders: build your own using the pattern in Step 3.

---

## Step 3 — Implement Consent Storage

```typescript
// lib/cookieConsent.ts
const CONSENT_VERSION = 1;
const CONSENT_KEY = 'cookie_consent';

interface ConsentState {
  version: number;
  analytics: boolean;
  marketing: boolean;
  grantedAt: string;
}

export function getConsent(): ConsentState | null {
  try {
    const raw = localStorage.getItem(CONSENT_KEY);
    if (!raw) return null;
    const consent = JSON.parse(raw) as ConsentState;
    if (consent.version !== CONSENT_VERSION) return null; // Force re-consent on version bump
    return consent;
  } catch {
    return null;
  }
}

export function grantConsent(analytics: boolean, marketing: boolean): void {
  localStorage.setItem(
    CONSENT_KEY,
    JSON.stringify({ version: CONSENT_VERSION, analytics, marketing, grantedAt: new Date().toISOString() })
  );
}

export function revokeConsent(): void {
  localStorage.removeItem(CONSENT_KEY);
}
```

---

## Step 4 — Gate Script Loading Behind Consent

```typescript
// app/layout.tsx (Next.js) or index.html
function loadAnalytics() {
  const consent = getConsent();
  if (!consent?.analytics) return;

  // Only load PostHog after consent
  import('posthog-js').then(({ default: posthog }) => {
    posthog.init(process.env.NEXT_PUBLIC_POSTHOG_KEY!, {
      api_host: '/ingest',
      loaded: (ph) => {
        if (process.env.NODE_ENV === 'development') ph.opt_out_capturing();
      },
    });
  });
}

// Call on page load and after consent is granted
loadAnalytics();
```

---

## Step 5 — Build the Consent Banner Component

```tsx
// components/CookieBanner.tsx
export function CookieBanner() {
  const [visible, setVisible] = useState(false);

  useEffect(() => {
    if (!getConsent()) setVisible(true);
  }, []);

  if (!visible) return null;

  function acceptAll() {
    grantConsent(true, true);
    setVisible(false);
    loadAnalytics();
  }

  function acceptNecessaryOnly() {
    grantConsent(false, false);
    setVisible(false);
  }

  return (
    <div role="dialog" aria-label="Cookie consent">
      <p>
        We use cookies to improve your experience. Analytics cookies help us understand usage.{' '}
        <a href="/privacy-policy">Privacy Policy</a>
      </p>
      <button onClick={acceptAll}>Accept all</button>
      <button onClick={acceptNecessaryOnly}>Necessary only</button>
    </div>
  );
}
```

---

## Step 6 — Add Consent Management to Settings

Allow users to change their preferences after the initial choice:

```tsx
// In settings/privacy section
<button onClick={() => { revokeConsent(); window.location.reload(); }}>
  Update cookie preferences
</button>
```

---

## Step 7 — Verify

- [ ] Visit the site in an incognito window — the banner appears
- [ ] Open browser DevTools → Network: no analytics scripts loaded before consent
- [ ] Click "Accept all" — analytics scripts load
- [ ] Revisit the site — banner does not appear again (consent persists)
- [ ] Open DevTools → Application → Local Storage: `cookie_consent` key is present
- [ ] Click "Update cookie preferences" → revoke → banner reappears on reload

---

## Related Documents

- [Compliance cookie consent](../compliance/cookie-consent.md)
- [Compliance GDPR](../compliance/gdpr.md)
