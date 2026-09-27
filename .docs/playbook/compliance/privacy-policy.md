---
kind: playbook
domain: compliance
topics: [compliance, privacy, policy]
authority: guidance
---

# Privacy Policy

A privacy policy is a legal requirement in most jurisdictions where you process personal data. It is not a formality — regulators and users consult it, and it must accurately reflect your actual data practices. A policy that describes practices you do not follow (or omits practices you do follow) exposes you to regulatory action.

## Required Disclosures

Every privacy policy must address these topics:

### What data you collect

List the categories of personal information collected, not just abstract categories. Examples:
- Account information: name, email address, password hash
- Billing information: billing address, last four digits of payment card, payment processor customer ID
- Usage data: pages visited, features used, timestamps of actions
- Device and technical data: IP address, browser type, operating system
- Communications: support messages, feedback submissions

### Why you collect it (purposes)

For each data category, state the purpose and the lawful basis (for GDPR). Examples:
- Email address: to authenticate the user and send transactional notifications (contract basis)
- Usage data: to improve the product and detect abuse (legitimate interest basis)
- Analytics cookies: to understand how users navigate the product (consent basis)

### How long you retain it

State retention periods per category:
- Account data: retained for the life of the account, then deleted or anonymized within 30 days of account deletion
- Server logs: 90 days
- Payment records: 7 years (legal/tax requirement)
- Anonymized analytics: 24 months

### Who you share it with

Name the categories of third parties (and ideally specific vendors):
- Payment processor: Stripe
- Email provider: SendGrid
- Error monitoring: Sentry
- Analytics: PostHog

For each, state what data is shared and the purpose.

### User rights

State the rights users can exercise, the mechanism to invoke them, and the response timeline:
- Access, rectification, erasure, portability (GDPR)
- Right to know, delete, opt-out (CCPA)
- How to submit a request: email address or in-product form
- Response timeline: 30 days (GDPR), 45 days (CCPA)

### Contact information

Provide a dedicated contact for privacy questions — typically a `privacy@` email address. If you have a Data Protection Officer (required for some EU organizations), provide their contact details.

### Policy update process

State how users will be notified of policy changes. Material changes should be communicated proactively (email notification, in-app banner) — not by silent update.

## Keeping the Policy Accurate

The most common privacy policy failure is drift: the policy describes practices that no longer match reality. Audit the policy when:
- Adding a new third-party integration that receives user data
- Starting to collect a new data category
- Changing the retention period for any category
- Removing a third-party vendor

A policy that says "we do not sell your data" when you have added an advertising pixel is a regulatory and reputational liability.

## Related Documents

- [GDPR compliance](gdpr.md)
- [CCPA compliance](ccpa.md)
- [Cookie consent](cookie-consent.md)
- [Terms of service](terms-of-service.md)
- [Security data privacy](../security/data-privacy.md)
