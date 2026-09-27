---
kind: playbook
domain: compliance
topics: [compliance, ccpa]
authority: guidance
---

# CCPA / CPRA

The California Consumer Privacy Act (CCPA), amended by the California Privacy Rights Act (CPRA), grants California residents rights over their personal information. It applies to for-profit businesses meeting at least one of these thresholds:

- Annual gross revenue over $25 million
- Annually buy, sell, receive, or share the personal information of 100,000 or more California consumers or households
- Derive 50% or more of annual revenue from selling or sharing California consumers' personal information

Even below these thresholds, implementing the key rights is low-cost and positions the product well for future scale.

## Key Rights

### Right to Know (Access)

California residents have the right to know:
- What categories of personal information you collect
- The purposes for which it is used
- The categories of third parties with whom it is shared
- The specific personal information collected about them (upon verifiable request)

Respond to verified access requests within 45 days.

### Right to Delete

Consumers can request that you delete their personal information. Unlike GDPR's right to erasure, CCPA allows more exceptions (e.g., completing a transaction, detecting security incidents). Still, implement deletion as hard delete or anonymization — not soft delete — for the same reasons as GDPR.

### Right to Opt-Out of Sale or Sharing

If you sell or share personal information (including sharing with advertising networks for behavioral advertising), you must provide a "Do Not Sell or Share My Personal Information" link on your homepage and in your privacy policy. Most SaaS products do not sell personal data; if you run targeted advertising, you likely do "share" it under CCPA's expanded definition.

### Right to Correct

Consumers can request correction of inaccurate personal information.

### Right to Limit Use of Sensitive Personal Information

Consumers can limit how you use sensitive categories (social security numbers, financial info, health data, geolocation, etc.) to only what is necessary to provide the service.

## How CCPA Differs from GDPR

| Dimension | GDPR | CCPA/CPRA |
|---|---|---|
| Scope | Any organization serving EU residents | For-profit businesses meeting California thresholds |
| Basis for rights | Applies by default; no threshold | Applies to businesses above revenue/volume thresholds |
| Opt-in vs. opt-out | Requires opt-in consent for non-essential processing | Opt-out model (processing allowed unless consumer objects) |
| Breach notification | 72 hours to supervisory authority | 30 days to attorney general; prompt notice to consumers |
| Enforcement | Supervisory authority fines up to 4% of global revenue | California AG + Private right of action for data breaches |

## Implementation Checklist

- [ ] Privacy policy lists data categories, purposes, third-party sharing, and consumer rights
- [ ] "Do Not Sell or Share My Personal Information" link if you share data for advertising
- [ ] Verifiable request mechanism for access and deletion (email request + identity verification)
- [ ] Process deletion requests as hard delete / anonymization within 45 days
- [ ] Honor opt-out signals (Global Privacy Control browser setting) if sharing data for advertising

## Related Documents

- [GDPR compliance](gdpr.md)
- [Privacy policy](privacy-policy.md)
- [Security data privacy](../security/data-privacy.md)
