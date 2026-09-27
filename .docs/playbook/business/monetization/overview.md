---
kind: playbook
domain: business
topics: [business, monetization]
authority: guidance
---

# Monetization Overview

The monetization model determines how the product captures value from the users it creates value for. Getting this right is as important as product design — the wrong model can doom a product with genuine value by pricing it in a way users won't accept or that doesn't sustain the business.

## Model Comparison

| Model | Charge structure | Fits when | Avoid when |
|---|---|---|---|
| **Flat subscription** | Fixed monthly/annual fee | Clear, bounded value proposition; predictable cost for both sides | Value varies wildly by user — some users get 100× more value than others |
| **Seat-based** | Price per user per month | Team/collaboration product where value scales with team size | Solo users dominate; charging per seat punishes early adopters |
| **Usage-based** | Price per unit of consumption | Value is closely tied to usage volume (API calls, messages sent, storage consumed) | Usage is unpredictable and hard to meter; users resent surprise bills |
| **Freemium-to-paid** | Free tier + paid tier(s) | High natural virality; network effects; conversion from free to paid is measurable | The product is expensive to run per user at scale — the free tier becomes a cost center |
| **One-time purchase** | Single payment for perpetual access | Tools with clear, stable feature sets; no ongoing infrastructure cost | The product requires ongoing hosting, support, or updates that a one-time price won't sustain |

## How to Choose

The monetization model should match the pattern by which users experience value:

- **Value is delivered continuously** (ongoing service, data sync, notifications) → subscription
- **Value scales with team size** (collaboration features, per-seat limits) → seat-based
- **Value is consumed in discrete units** (AI tokens, API calls, email sends) → usage-based
- **Value is delivered once** (a template, a course, a tool that doesn't depend on ongoing service) → one-time purchase
- **Value has two speeds** (basic use is simple; heavy use justifies payment) → freemium

Cross-reference with the monetization model decision in [core/project-brief.md](../../product/project-brief.md) where this was first determined.

## MVP Pricing Principle

Keep pricing simple for the MVP. Two or three tiers are enough. Pricing complexity (many tiers, complex usage metering, add-ons) should be added only when you have enough paying users to know what they actually value and how they use the product.

## Related Documents

- [Pricing strategy](pricing-strategy.md)
- [Freemium design](freemium.md)
- [Trials design](trials.md)
- [Usage-based billing](usage-based.md)
- [One-time purchases](one-time-purchases.md)
- [Upgrade flows](upgrade-flows.md)
- [Core project brief](../../product/project-brief.md)
