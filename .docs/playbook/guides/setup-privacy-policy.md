---
kind: playbook
domain: guides
topics: [guides, setup, privacy, policy]
authority: guidance
---

# How to Setup a Privacy Policy

**What you'll have at the end:** A compliant privacy policy hosted at a stable URL, linked from signup and footer, with a review schedule.

---

## Step 1 — Audit Your Actual Data Practices

Before drafting anything, document what you actually do. Write down:

```
Data collected:
- Email address (required for account)
- Name (optional, collected at signup)
- Billing address (collected by Stripe at checkout)
- Usage events (page views, feature clicks — via PostHog if consented)
- IP addresses (in server logs, retained 30 days)

Data shared with:
- Stripe (payment processing)
- PostHog / analytics provider (if consented)
- SendGrid / email provider (to send transactional emails)
- Hosting provider (server infrastructure)

Data retention:
- Account data: retained while account is active + 30 days after deletion request
- Payment records: 7 years (tax requirement)
- Server logs: 30 days

User rights:
- Access: yes, via data export
- Deletion: yes, via account deletion in settings (hard delete + Stripe cancel)
- Correction: yes, via profile settings
```

---

## Step 2 — Draft the Policy

Use your audit from Step 1. The policy must cover every disclosure from [compliance/privacy-policy.md](../compliance/privacy-policy.md):

1. What personal data is collected and how
2. Why it is collected (legal basis under GDPR if serving EU users)
3. How long data is retained
4. Who data is shared with (third parties, processors)
5. User rights and how to exercise them
6. Contact information (email for privacy inquiries)
7. Policy last-updated date

**Generator tools** can give you a starting draft: Termly, iubenda, and GetTerms.io all produce reasonable policy text. Always review and edit to match your actual data practices — a template that doesn't match reality is worse than no policy.

---

## Step 3 — Host the Policy at a Stable URL

The policy must live at a URL that never changes. Two standard patterns:

- `yourdomain.com/privacy-policy`
- `yourdomain.com/legal/privacy`

Do not use versioned URLs like `/privacy-policy-v2` — update the content at the same URL and change the "last updated" date.

```tsx
// pages/privacy-policy.tsx or app/privacy-policy/page.tsx
export default function PrivacyPolicy() {
  return (
    <article>
      <h1>Privacy Policy</h1>
      <p><em>Last updated: January 1, 2025</em></p>
      {/* Policy content */}
    </article>
  );
}
```

---

## Step 4 — Link the Policy

Link from every required location:

**Signup form:**
```tsx
<p>
  By creating an account, you agree to our{' '}
  <a href="/terms-of-service">Terms of Service</a> and{' '}
  <a href="/privacy-policy">Privacy Policy</a>.
</p>
```

**Footer (every page):**
```tsx
<footer>
  <a href="/privacy-policy">Privacy Policy</a>
  <a href="/terms-of-service">Terms of Service</a>
</footer>
```

**Cookie consent banner:**
```tsx
<p>
  We use cookies to improve your experience.{' '}
  <a href="/privacy-policy">Learn more</a>
</p>
```

---

## Step 5 — Set a Review Schedule

Privacy policies drift from reality as the product evolves. Set a calendar reminder to review the policy:

- Quarterly for fast-moving products
- Bi-annually for stable products
- Immediately after adding any new third-party data processor

When the policy changes materially, notify existing users by email before the change takes effect.

---

## Verify

- [ ] Privacy policy is accessible at the stable URL without logging in
- [ ] The URL is linked from the signup page
- [ ] The URL is linked from the footer on every page
- [ ] The URL is linked from the cookie consent banner
- [ ] Policy content matches current data practices
- [ ] "Last updated" date is current
- [ ] A review reminder is on the calendar

---

## Related Documents

- [Compliance privacy policy](../compliance/privacy-policy.md)
- [Compliance GDPR](../compliance/gdpr.md)
- [How to setup cookie consent](setup-cookie-consent.md)
