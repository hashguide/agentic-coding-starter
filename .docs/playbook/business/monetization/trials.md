---
kind: playbook
domain: business
topics: [business, monetization, trials]
authority: guidance
---

# Free Trials

A free trial gives users temporary access to paid features to evaluate the product. Unlike freemium, it's time-limited — the user either converts to paid or loses access. The design of the trial (type, length, credit card requirement) significantly impacts conversion rates.

## Trial Types

### Time-Limited with Full Access

The user gets full access to all paid features for a defined period (typically 7–14 days). No restrictions on features or usage volume during the trial.

**Best for:** Products with clear, demonstrable value that users can experience quickly; B2B products where evaluation requires full feature access; products with high natural activation rates.

**Tradeoffs:** Works only if users actually get to the "aha moment" within the trial period. A 7-day trial for a product that requires 5 days of setup produces 2 days of evaluation.

### Usage-Limited (No Time Cap)

The user gets limited usage of a feature rather than time-limited access — e.g., "first 10 AI generations are free" or "first 1,000 API calls included."

**Best for:** API products or usage-based products where the value is clearly per-unit; products with users who adopt at different speeds.

**Tradeoffs:** Users who use the product infrequently aren't time-pressured. May result in lower conversion urgency but better-quality trial users.

### Combined: Time + Usage

Some products combine both — a 14-day time limit AND a usage cap. The trial ends at whichever limit is reached first.

## Credit Card Required vs. Not

| Approach | Conversion to start | Conversion to paid | Best for |
|---|---|---|---|
| **No credit card required** | Higher — lower friction to start | Lower — no payment intent signal | Products where self-service trial starts fast; consumer / developer tools |
| **Credit card required** | Lower — card entry friction | Higher — cards charged automatically at trial end | B2B products with deliberate evaluation process; higher ACV products |

**Recommendation:** No credit card required is the default for most self-serve SaaS. The lower barrier to starting the trial is worth the lower payment conversion rate when multiplied across a large volume of trial starts. Credit card at trial start makes sense when each trial start has significant sales-assist cost.

## Trial Length

| Trial length | Appropriate for |
|---|---|
| 7 days | Simple tools where value is demonstrated in a single session |
| 14 days | Products requiring a week of real use to evaluate properly |
| 30 days | Complex products with longer evaluation cycles; B2B products evaluated by committees |

**A shorter trial with an extension option is better than a long trial with no urgency.** A 7-day trial that can be extended to 14 days by contacting support produces better conversion than a 30-day trial where no conversion urgency ever develops.

## Trial-to-Paid Email Sequence

At minimum, send:
1. **Day 0:** Welcome — what to do first to get value
2. **Day 3:** "Are you set up?" — nudge toward activation if they haven't completed the core action
3. **Day N-3 (3 days before expiry):** "Your trial ends in 3 days" — conversion prompt with a link to upgrade
4. **Day N (trial expiry):** "Your trial has ended" — offer to upgrade or extend

## Related Documents

- [Monetization overview](overview.md)
- [Freemium](freemium.md)
- [Upgrade flows](upgrade-flows.md)
- [Email transactional](../../integrations/email/transactional.md)
