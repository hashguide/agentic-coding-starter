---
kind: playbook
domain: experience
topics: [experience, journeys, new, user]
authority: guidance
---

# New User Flow

The new user flow covers the complete journey from first visit to becoming an activated user. Each sub-flow has a distinct goal and a set of design rules.

## Sub-Flow 1: First Visit / Landing Page

**Goal:** Visitor understands what the product does and for whom, and takes one step toward account creation.

**Design rules:**
- The hero section answers "what is this for me?" in one headline and one subheadline
- One primary CTA per visible section
- Social proof appears early (logos, testimonials, user counts) — visitors make trust decisions in the first few seconds
- The path to signup is never more than one click from the landing page

**Success state:** Visitor clicks the primary CTA and reaches the signup form.

## Sub-Flow 2: Signup / Registration

**Goal:** Visitor creates an account with minimum friction.

**Design rules:**
- Ask for email + password only (or social login) — defer all other information to onboarding
- Password reset and "already have an account" links must be present on the signup page
- If email is already registered, send a "you already have an account" email and show the same "check your email" response as a new signup (no enumeration)
- Email verification is acceptable as a gate but provide grace-period access to core features while awaiting verification

**Success state:** Account created. User is authenticated and redirected to onboarding or the dashboard.

## Sub-Flow 3: Onboarding

**Goal:** User takes the first core action and experiences the product's primary value proposition.

**Design rules:**
- Guide the user toward the single most important action (the "aha moment" action)
- Every step is skippable — never hard-gate core product access behind onboarding completion
- The wizard or checklist has no more than 4–5 steps
- Sample data or a template is provided if the product starts empty — an empty state is a poor first experience
- Progress is visible (progress bar, step indicator, checklist completion)

**Success state:** User has completed the core first action (created a project, sent an invoice, connected an integration — whatever the product's primary value action is).

## Sub-Flow 4: Empty State (First Session)

**Goal:** User understands what to do next and takes their second meaningful action.

**Design rules:**
- The first-time empty dashboard is not the default "no data" empty state — it is an onboarding prompt disguised as an empty state
- Show the recommended first action as a prominent CTA, not as a tiny text link
- Do not show all empty states at once (notifications empty + projects empty + activity empty) — prioritize one call to action

**Success state:** User has created a second artifact or taken a second action that demonstrates they understand the core workflow.

## Sub-Flow 5: Paywall / Paid Feature Encounter

**Goal:** Free user understands the value of upgrading and takes a step toward becoming a paying customer.

**Design rules:**
- Users should interact with and derive value from the product before encountering a paywall — non-restrictive freemium framing
- When a user hits a paid feature, show a preview or explanation of what the feature does before asking for payment
- The upgrade prompt includes a clear statement of what the user gets and the price
- Offer a trial before requiring payment information if the product supports it

**Success state:** User starts a trial or upgrades. If they dismiss the paywall, they are not blocked from continuing to use available free features.

## Related Documents

- [Flow goals](flow-goals.md)
- [Returning user flow](returning-user.md)
- [Onboarding layout](../layouts/onboarding.md)
- [Pricing layout](../layouts/pricing.md)
- [Monetization trials](../../business/monetization/trials.md)
