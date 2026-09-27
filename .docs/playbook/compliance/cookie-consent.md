---
kind: playbook
domain: compliance
topics: [compliance, cookie, consent]
authority: guidance
---

# Cookie Consent

Cookies (and equivalent tracking technologies like pixels, fingerprinting, and local storage identifiers) require informed consent under GDPR and the ePrivacy Directive for EU users. Consent must be obtained before non-essential scripts load — not after.

## Cookie Categories

Classify every cookie your product sets into one of these categories:

| Category | Requires consent | Examples |
|---|---|---|
| **Strictly necessary** | No — these are required for the service to function | Session cookies, CSRF tokens, auth state |
| **Functional** | Depends — if necessary for user-chosen features | Language preference, saved UI state |
| **Analytics** | Yes | Google Analytics, Mixpanel, PostHog, Amplitude |
| **Marketing / Advertising** | Yes | Meta Pixel, Google Ads conversion tracking |

Most SaaS products use strictly necessary cookies (session management) and analytics cookies. The strictly necessary cookies need no consent — they are required to provide the service. Analytics cookies require explicit opt-in for EU users.

## Consent Banner UX

A compliant consent banner for EU users:

- Presents choices clearly without dark patterns (no pre-ticked boxes, no "Accept all" prominently styled vs. buried "Reject all")
- Offers a genuine way to decline without losing access to the service
- Presents options per category — not just "accept all or leave"
- Makes it as easy to withdraw consent as to give it

**Do not:**
- Grey out or hide the "Reject all" or "Decline" option
- Use misleading labels like "Privacy settings" instead of "Cookie preferences"
- Load analytics or marketing scripts before consent is recorded

## Consent Storage

Store the user's consent decision so you don't re-prompt on every page load:

```typescript
type ConsentPreferences = {
  analytics: boolean;
  marketing: boolean;
  timestamp: number;
  version: string; // bump when cookie policy changes to re-prompt
};

function saveConsent(prefs: ConsentPreferences): void {
  localStorage.setItem('cookie_consent', JSON.stringify(prefs));
}

function loadConsent(): ConsentPreferences | null {
  const raw = localStorage.getItem('cookie_consent');
  if (!raw) return null;
  const prefs = JSON.parse(raw) as ConsentPreferences;
  // Re-prompt if policy version has changed
  if (prefs.version !== CURRENT_POLICY_VERSION) return null;
  return prefs;
}
```

Re-prompt when your cookie policy changes materially (new tracking vendors, new purposes).

## Conditional Script Loading

Only load analytics or marketing scripts after consent is granted:

```typescript
function initAnalytics(): void {
  const consent = loadConsent();
  if (!consent?.analytics) return;

  // Load PostHog only after consent
  import('posthog-js').then(({ default: posthog }) => {
    posthog.init(process.env.NEXT_PUBLIC_POSTHOG_KEY!, {
      api_host: 'https://app.posthog.com',
      loaded: (ph) => {
        if (!consent.analytics) ph.opt_out_capturing();
      },
    });
  });
}
```

## Security

**Loading analytics or marketing scripts before consent is a legal violation in GDPR jurisdictions.** The ePrivacy Directive requires prior informed consent before storing or accessing cookies for non-essential purposes. Scripts that fire on page load before the consent banner renders violate this requirement.

Additionally, loading third-party scripts unconditionally is a supply-chain risk: a compromised analytics vendor can exfiltrate form data. Conditional loading limits this exposure to consenting users only.

## Related Documents

- [GDPR compliance](gdpr.md)
- [Privacy policy](privacy-policy.md)
- [Security overview](../security/overview.md)
