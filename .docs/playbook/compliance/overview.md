---
kind: playbook
domain: compliance
topics: [compliance]
authority: guidance
---

# Compliance Overview

Compliance requirements depend on who you serve and how you process their data. Most SaaS products are subject to at least one of these frameworks — knowing which ones apply before launch prevents retroactive scrambles to retrofit controls.

## Which Rules Apply to You

Answer these questions:

**Do you serve users in the European Union or UK?**
→ GDPR applies. This covers data subject rights, lawful basis for processing, Data Processing Agreements with vendors, and 72-hour breach notification. See [compliance/gdpr.md](gdpr.md).

**Do you serve users in California?**
→ CCPA/CPRA applies if you meet certain thresholds (gross annual revenue > $25M, buy/sell/receive/share data of ≥ 100,000 consumers, or derive ≥ 50% of revenue from selling personal data). Even below the threshold, implementing CCPA controls is low-cost and builds trust. See [compliance/ccpa.md](ccpa.md).

**Do you process payments (card numbers)?**
→ PCI DSS scope considerations apply. Using a tokenizing library (Stripe.js/Elements) keeps raw card data off your servers entirely and reduces scope to SAQ A — the minimal tier. See [compliance/pci-dss.md](pci-dss.md).

**Do you use cookies, tracking pixels, or analytics?**
→ Cookie consent requirements apply under GDPR (and ePrivacy Directive) for EU users, and similar rules apply in other jurisdictions. See [compliance/cookie-consent.md](cookie-consent.md).

**Is your product publicly accessible?**
→ ADA (US) and the European Accessibility Act (EU) require WCAG 2.1 AA compliance. See [compliance/accessibility-compliance.md](accessibility-compliance.md).

## Decision Summary

| Condition | Framework | Key requirement |
|---|---|---|
| Serve EU/UK users | GDPR | Lawful basis, data subject rights, DPAs, 72-hour breach notice |
| Serve California users (above threshold) | CCPA/CPRA | Right to know, right to delete, opt-out of sale/sharing |
| Process card payments | PCI DSS | Use client-side tokenization; never handle raw card numbers |
| Use cookies/analytics | Cookie consent (GDPR, ePrivacy) | Obtain consent before loading non-essential scripts |
| Publicly accessible product | ADA / EAA | WCAG 2.1 AA accessibility |

## Documents in This Directory

- [gdpr.md](gdpr.md) — Lawful basis, data subject rights, DPAs, breach notification
- [ccpa.md](ccpa.md) — California residents' rights, opt-out, disclosure requirements
- [cookie-consent.md](cookie-consent.md) — Cookie categories, consent banners, consent storage
- [privacy-policy.md](privacy-policy.md) — Privacy policy content requirements
- [terms-of-service.md](terms-of-service.md) — ToS content requirements
- [accessibility-compliance.md](accessibility-compliance.md) — ADA, WCAG 2.1 AA, EAA
- [data-retention.md](data-retention.md) — Retention policies and automated deletion
- [pci-dss.md](pci-dss.md) — PCI scope reduction via client-side tokenization

## Related Documents

- [Security data privacy](../security/data-privacy.md)
- [Security checklist](../security/checklist.md)
- [Database schema design](../data/schema-design.md)
