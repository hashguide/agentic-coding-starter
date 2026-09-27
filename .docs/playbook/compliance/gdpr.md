---
kind: playbook
domain: compliance
topics: [compliance, gdpr]
authority: guidance
---

# GDPR

The General Data Protection Regulation applies to any organization that processes personal data of EU or UK residents, regardless of where the organization is based. For a SaaS product with EU users, GDPR is not optional.

## Lawful Bases for Processing

Processing personal data requires a lawful basis. Six bases exist; the three most relevant to SaaS:

| Basis | When to use | Example |
|---|---|---|
| **Contract** | Processing is necessary to perform the service the user signed up for | Storing email to send login links; storing payment info to process a subscription |
| **Consent** | Processing is not required for the service; user has opted in clearly | Sending marketing emails; enabling analytics beyond basic product telemetry |
| **Legitimate interest** | You have a genuine business interest that doesn't override users' rights | Security logging; fraud prevention; product analytics with proper safeguards |

**Do not default to consent for everything.** If processing is necessary to deliver the service (contract basis), claiming consent implies the user can withdraw it and you'll stop processing — which is not true if the data is essential to the service. Use the correct basis.

## Data Subject Rights

GDPR grants users enforceable rights over their personal data:

| Right | What it requires |
|---|---|
| **Access (Art. 15)** | Provide a copy of all personal data you hold about the user, within 30 days |
| **Rectification (Art. 16)** | Correct inaccurate personal data |
| **Erasure (Art. 17)** | Permanently delete or anonymize all personal data when the user requests it — a soft-deleted record does NOT satisfy this right |
| **Portability (Art. 20)** | Provide data in a structured, commonly-used, machine-readable format (JSON, CSV) |
| **Restriction (Art. 18)** | Stop processing (but not delete) data while a dispute is pending |
| **Object (Art. 21)** | Object to processing based on legitimate interest; must stop if no compelling overriding reason |

Implement the access and erasure rights first — they are the most commonly exercised.

**Erasure explicitly:** a `deleted_at` timestamp is not erasure. The data is still in the database. Erasure means the user's PII is either deleted from every table or fully replaced with non-identifying values. See [security/data-privacy.md](../security/data-privacy.md) and [database/schema-design.md](../data/schema-design.md) for implementation.

## Data Processing Agreements (DPAs)

When you use a third-party service that processes personal data on your behalf, that service is a data processor and you must have a DPA in place. This includes:

- Email providers (SendGrid, Postmark, Resend)
- Analytics platforms (Mixpanel, Amplitude, PostHog)
- Payment processors (Stripe)
- Customer support platforms (Intercom, Crisp)
- Error monitoring (Sentry)
- Cloud infrastructure providers (AWS, GCP, Hetzner)

Most of these providers offer a standard DPA that you accept via their settings page or by ticking a checkbox at signup. Do this — it is a legal requirement to have one in place, not a nice-to-have.

## Breach Notification

If a personal data breach occurs (unauthorized access, accidental disclosure, loss of data), you must:

1. Notify the relevant supervisory authority within **72 hours** of becoming aware
2. Notify affected individuals **without undue delay** if the breach is likely to result in a high risk to their rights

Document the breach even if you decide notification is not required — regulators may request the record.

## Security

Data minimization and purpose limitation are legal requirements, not best practices. You must:
- Collect only personal data that is necessary for the stated purpose
- Use personal data only for the purpose it was collected under
- Not retain data longer than necessary

A data category that you cannot justify collecting for a specific purpose must not be collected.

## Related Documents

- [CCPA compliance](ccpa.md)
- [Security data privacy](../security/data-privacy.md)
- [Database schema design (soft/hard delete)](../data/schema-design.md)
- [Data export and deletion how-to](../guides/setup-data-export-and-deletion.md)
- [Privacy policy](privacy-policy.md)
