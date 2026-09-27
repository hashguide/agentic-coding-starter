---
kind: playbook
domain: experience
topics: [experience, layouts, onboarding]
authority: guidance
---

# Onboarding Layout

Onboarding is the bridge between account creation and the moment the user first experiences value. A well-designed onboarding flow reduces time-to-value and increases activation (the percentage of users who complete the "aha moment" action). Every step that is not leading directly to value is a potential drop-off point.

## Three Common Onboarding UI Patterns

### Progress Checklist

A list of tasks the user should complete, displayed persistently (often in the dashboard or a sidebar widget). Each item can be completed in any order.

```
Getting started                    2/5 complete
━━━━━━━━━━━━━━━━░░░░░░░░░░░░░░░░

✓ Create your account
✓ Verify your email
○ Create your first project    ← current recommendation
○ Invite a team member
○ Connect your first integration
```

**When to use:** Products where onboarding tasks are largely independent and can be done in any order. Good for B2B tools where setup is spread over days. The checklist persists until all items are complete, then disappears from the dashboard.

**Tradeoffs:** Flexible but low guidance — users have to decide where to start. Works best when each task is self-contained and the user already understands what the product does.

### Step-by-Step Wizard

A multi-step modal or full-page flow that guides the user through a linear sequence of setup steps.

```
[Step 1 of 4] Set up your workspace

[Form: workspace name]
[Form: invite team members]

[Skip for now]   [Continue →]
```

**When to use:** Products with required setup that must happen before the core workflow is accessible (connecting an integration, configuring a data source, choosing a plan). The linear sequence prevents the user from getting to an empty, non-functional state.

**Tradeoffs:** High guidance but potentially long. Every additional step increases drop-off. Keep the wizard to 2–4 steps maximum. Anything more complex should be broken into progressive disclosure (show core wizard, reveal advanced setup later).

### Interactive Product Tour

An overlay-based tour that guides the user through the existing interface, highlighting elements and explaining what they do.

```
[Product UI with overlay backdrop]

[Tooltip anchored to the "New Project" button]:
"Start here — create your first project."
[Next →]   [Skip tour]
```

**When to use:** Products where the UI itself tells the story if the user knows where to look. Works best for products with an existing workspace or sample data that demonstrates value immediately.

**Tradeoffs:** Overlay tours are often dismissed quickly and rarely re-viewed. Best combined with sample/demo data so the user is exploring a real-looking environment, not an empty one.

## Every Onboarding Step Must Be Skippable

No onboarding step should be a hard gate that blocks product access. Every step has a "Skip" or "Skip for now" option.

**Why:** Forcing completion of steps before a user can reach the product creates resentment and drop-off. Users who skip setup can always return to it (via the checklist in the dashboard, or the settings page). Users who are blocked by a required step may not return at all.

**Exception:** Email verification — a verification step that requires confirming the email address before full access is a common and accepted gate. Even here, a "Resend verification email" and grace-period access to core features reduces friction.

## Related Documents

- [Onboarding copy](../content/onboarding-copy.md)
- [New user UX flow](../journeys/new-user.md)
- [Dashboard layout (checklist widget)](dashboard.md)
- [Empty states (first-time empty)](../interaction/empty-states.md)
