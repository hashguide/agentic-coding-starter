---
kind: playbook
domain: business
topics: [business, monetization, upgrade, flows]
authority: guidance
---

# Upgrade Flows

An upgrade prompt appears when a user attempts to use a feature or reach a volume that requires a paid plan. The design of that prompt — how it's framed, where it appears, and what options it offers — determines whether the user upgrades, downgrades their expectations, or leaves.

## Frame Around Gain, Not Restriction

The worst upgrade prompts are walls: "You've hit your limit. Upgrade to continue." This framing is punitive — you've taken something away.

Better prompts frame around what the user gains:

| Punitive (avoid) | Gain-framed (use) |
|---|---|
| "You've reached the limit of 3 projects on the free plan." | "Upgrade to Pro to create unlimited projects — and get [other benefit]." |
| "This feature is not available on your plan." | "AI-powered suggestions are available on the Pro plan. Upgrade to start using them." |
| "Your trial has ended. You must upgrade to continue." | "Your trial is over. Keep everything you've built by upgrading to Pro." |

The difference: gain-framed prompts tell the user what they get. Punitive prompts tell them what they've lost.

## Placement: Inline, Not Redirect

Upgrade prompts must appear inline at the point of friction — not as a full-page redirect to a separate pricing page.

**Wrong:** User clicks "Create project" → redirected to `/pricing` page → user has to navigate back or abandon.

**Right:** User clicks "Create project" → a contextual prompt appears inline (modal or banner) → user can upgrade without leaving context, or dismiss and continue with what they have.

Inline prompts are dismissible. The user should always be able to close the prompt and return to what they were doing.

## Worked Examples

### Well-Framed Upgrade Prompt

```tsx
function UpgradeModal({ feature, onClose, onUpgrade }: UpgradeModalProps) {
  return (
    <Modal open onClose={onClose}>
      <div className="upgrade-prompt">
        <SparklesIcon className="upgrade-icon" aria-hidden="true" />
        <h2>Unlock unlimited projects</h2>
        <p>
          You've built 3 great projects — you clearly know what you're doing.
          Upgrade to Pro to create as many as you need, plus get access to
          team collaboration, priority support, and advanced analytics.
        </p>
        <div className="upgrade-pricing">
          <strong>Pro — $29/month</strong>
          <span>or $24/month, billed annually</span>
        </div>
        <button onClick={onUpgrade} className="btn-primary">
          Upgrade to Pro
        </button>
        <button onClick={onClose} className="btn-ghost">
          Stay on free plan
        </button>
      </div>
    </Modal>
  );
}
```

### Poorly-Framed Upgrade Prompt (Do Not Use)

```
You have reached the maximum of 3 projects allowed on the Free plan.
Please upgrade your plan to create additional projects.

[Upgrade]  [Cancel]
```

**Problems:** Punitive framing ("maximum allowed"), no benefit statement, no social proof, dismissing feels like failing.

## When the Upgrade Moment Should Not Be a Prompt

Some "upgrade" moments should be previews, not blockers:

- A feature marked as "Pro" but visible to free users — clicking it shows a preview with the upgrade CTA
- A dashboard widget that is grayed out with a lock icon and a tooltip: "Available on Pro — upgrade to unlock"

Previews are less intrusive than blocking prompts and are appropriate for features the user hasn't tried to use yet.

## Related Documents

- [Monetization overview](overview.md)
- [Freemium](freemium.md)
- [Pricing strategy](pricing-strategy.md)
- [Pricing layout](../../experience/layouts/pricing.md)
