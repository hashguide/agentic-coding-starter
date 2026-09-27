---
kind: playbook
domain: product
topics: [product, type, selection]
authority: guidance
---

# Product Type Selection

The product type determines which parts of this playbook matter most. A hosted subscription application is only one possibility: the same planning and engineering practices also apply to mobile and desktop apps, marketplaces, e-commerce stores, APIs, command-line tools, libraries, content products, and internal tools. Classify the product before choosing architecture, distribution, and monetization.

## Start With the Value Delivery Model

Describe how value reaches the user, not which framework you expect to use. A product may combine multiple types—for example, a marketplace can have a web storefront, mobile seller app, and public API—but one type should define the first release.

| Product type | Value delivery | Primary constraint | Common revenue model |
|---|---|---|---|
| Hosted web application | Continuously through a browser | Availability, security, operations | Subscription or usage-based |
| Mobile application | Installed on a personal device | Store review, device permissions, offline behavior | Paid download, in-app purchase, subscription |
| Desktop application | Installed on a computer | Packaging, updates, OS integration | Perpetual license, paid upgrades, subscription |
| Marketplace | Exchange between two or more participant groups | Liquidity, trust, payouts, disputes | Commission, listing fee, subscription |
| E-commerce product | Physical or digital goods are sold directly | Inventory, fulfillment, returns, tax | Per-item margin or bundles |
| API or developer platform | Capability consumed by software | Stable contracts, quotas, developer experience | Usage-based, tiered access |
| CLI, library, or open-source project | Capability integrated into a workflow or codebase | Compatibility, documentation, release discipline | License, support, sponsorship, hosted service |
| Content or community product | Information, education, or access | Editorial quality, moderation, discovery | One-time sale, membership, advertising |
| Internal tool | Operational efficiency for a known organization | Workflow fit, permissions, maintainability | Cost savings rather than direct revenue |

## Decision Questions

1. Does value require an always-on server, or can it be delivered by an installed artifact or download?
2. Is the buyer also the user? Marketplaces, education, and enterprise tools often have several stakeholders.
3. Who controls distribution: you, an app store, a package registry, a marketplace, or an employer?
4. Is value delivered once, continuously, or in measurable units?
5. What must continue working offline, and which data belongs on the device?
6. Are payments for your own product, goods sold by others, or both?

Record the answers in the project brief. Do not add subscriptions, accounts, multi-tenancy, or a server merely because they are familiar defaults.

## How the Type Changes the Plan

| Decision area | Questions introduced by product type |
|---|---|
| Architecture | Web deployment, native packaging, offline storage, public API compatibility, or extension sandbox? |
| Access | Anonymous use, local license, individual account, organization membership, or marketplace roles? |
| Payments | Recurring billing, one-time checkout, in-app purchase, seller payouts, or no payment? |
| Compliance | Personal data, card payments, minors, user-generated content, app-store rules, or sector regulation? |
| Testing | Browsers, device/OS matrix, package consumers, API versions, payment flows, or fulfillment? |
| Launch | Website, app stores, package registries, enterprise rollout, communities, or channel partners? |

## Recommended Default

Choose the smallest delivery surface that proves the core value. A static site can validate content; a CLI can validate a developer workflow; a local desktop utility can avoid premature accounts and hosting; a web application can validate collaboration. Add platforms only after evidence shows that their distribution or capabilities are necessary.

## Related Documents

- [Project brief](project-brief.md)
- [Non-functional requirements](non-functional-requirements.md)
- [Architecture overview](../architecture/overview.md)
- [Monetization overview](../business/monetization/overview.md)
- [Distribution channels](../growth/launch/distribution-channels.md)
