---
kind: playbook
domain: experience
topics: [experience, layouts, pricing]
authority: guidance
---

# Pricing Page

The pricing page converts interested visitors into paying customers. It must answer the user's implicit questions: "What do I get?", "Is this worth it?", and "Which plan is right for me?" — in that order, without making them do the work to figure it out.

## Tier Comparison Layout

Use 2–3 tiers for an MVP pricing page. More than three tiers overwhelms the comparison and creates decision paralysis.

```
┌─────────────────────────────────────────────┐
│              Simple, fair pricing           │
│    [Monthly]    Annual (save 20%)   ←toggle │
├──────────────┬──────────────┬───────────────┤
│   Starter    │  Pro ★       │  Enterprise   │
│   $0 / mo   │  $29 / mo   │  Contact us   │
│              │  RECOMMENDED │               │
│ ─────────── │ ────────────│ ──────────── │
│ ✓ Feature 1 │ ✓ Everything │ ✓ Everything  │
│ ✓ Feature 2 │   in Starter │   in Pro      │
│ ✗ Feature 3 │ ✓ Feature 3 │ ✓ SSO         │
│ ✗ Feature 4 │ ✓ Feature 4 │ ✓ SLA         │
│             │             │ ✓ Dedicated   │
│ [Get started│ [Start trial]│   support     │
│   free]     │              │ [Contact us]  │
└──────────────┴──────────────┴───────────────┘
```

**Recommended tier:** Visually highlight the plan you want most users to choose — slightly larger card, colored border or header, "Most popular" or "Recommended" badge. This is the anchor for the purchasing decision.

**Feature comparison:** List what each tier includes. For higher tiers, either list all features (with higher-tier additions highlighted) or list "Everything in [lower tier], plus:". The second pattern is more readable.

## Monthly / Annual Billing Toggle

A toggle between monthly and annual billing should be visible near the top of the pricing table:

```tsx
function BillingToggle({ billing, onChange }: { billing: 'monthly' | 'annual'; onChange: (b: 'monthly' | 'annual') => void }) {
  return (
    <div className="billing-toggle">
      <button
        onClick={() => onChange('monthly')}
        className={cn('toggle-option', billing === 'monthly' && 'active')}
      >
        Monthly
      </button>
      <button
        onClick={() => onChange('annual')}
        className={cn('toggle-option', billing === 'annual' && 'active')}
      >
        Annual
        <span className="badge-discount">Save 20%</span>
      </button>
    </div>
  );
}
```

When annual is selected, show the monthly equivalent price ("$24/mo, billed annually as $288") rather than the annual total — most users think in monthly terms.

## FAQ Section

A brief FAQ section addresses the top objections to purchase. Common questions:

- **"What happens when my trial ends?"** — Answers transition anxiety
- **"Can I cancel anytime?"** — Answers commitment anxiety
- **"What payment methods do you accept?"** — Practical
- **"Can I change plans later?"** — Addresses "am I picking the wrong one"
- **"Do you offer refunds?"** — Addresses risk
- **"Is there a discount for non-profits / students?"** — Common segment

Keep FAQ answers short (2–3 sentences each). Link to the full terms/refund policy for users who need more detail.

## CTA Placement

- Primary CTA at the bottom of each plan card
- Repeat the most prominent CTA below the FAQ section
- Make trial CTAs clearly free ("Start free trial", "Try free for 14 days") to remove the fear of accidentally being charged

## Related Documents

- [Monetization pricing strategy](../../business/monetization/pricing-strategy.md)
- [Monetization trials](../../business/monetization/trials.md)
- [Upgrade flows](../../business/monetization/upgrade-flows.md)
- [Landing page layout](landing-page.md)
