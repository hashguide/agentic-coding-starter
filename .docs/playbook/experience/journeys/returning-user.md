---
kind: playbook
domain: experience
topics: [experience, journeys, returning, user]
authority: guidance
---

# Returning User Flow

Returning users have context, habits, and expectations. They logged in before, started work, and came back. The returning user flow covers the sub-flows that happen after the first session.

## Sub-Flow 1: Session Recovery / Login

**Goal:** Authenticated user arrives in the product in the right context quickly.

**Design rules:**
- Login form is minimal — email + password + "Forgot password?" link
- After login, redirect to the user's last active context, not always the home dashboard
- "Remember me" or persistent sessions reduce re-authentication friction for low-security use cases (be explicit about session duration)
- Error message for failed login is always "Incorrect email or password." — never distinguishes between email not found and wrong password

**Success state:** User is authenticated and lands in the correct context.

**Context restoration note:** If the user had /projects/123 open before their session expired, redirect there after login (via a `?next=` parameter in the login URL, validated server-side). Do not redirect to an absolute URL — validate the `next` parameter is a relative path on your own domain to prevent open redirect attacks.

## Sub-Flow 2: The Daily Core Loop

**Goal:** Active user completes their primary recurring action without friction.

**Design rules:**
- The dashboard surfaces pending actions, not generic stats — returning users know the stats; they want to know what needs attention
- The path to the most common action should be reachable in ≤ 2 clicks from any page
- Returning users should not see onboarding prompts they've already dismissed
- Save in-progress work automatically where possible; prompt before discarding unsaved changes

**Success state:** User completes their core action (reviews a report, sends an invoice, publishes content) and the product state is updated.

## Sub-Flow 3: Account / Billing Self-Service

**Goal:** User updates their plan, payment method, or account information without contacting support.

**Design rules:**
- All billing management is accessible from the settings page without opening a support ticket
- Plan upgrade and downgrade happen immediately with prorated billing
- Invoice history is downloadable
- Cancellation is possible from the settings page — do not hide it or require a phone call

**Success state:** User has updated their plan, payment method, or profile and received confirmation.

**Cancellation off-ramp:** Before cancellation is confirmed, show a simple retention offer (discount, pause option) — one screen, one offer, no dark patterns. Then let them cancel if they choose.

## Sub-Flow 4: Notification-Triggered Re-Entry

**Goal:** User who received a notification (email or in-app) arrives at the specific relevant context that triggered the notification.

**Design rules:**
- Every notification link goes directly to the specific record or action — not to the home dashboard
- Email notifications include a link that authenticates the session if possible (magic link approach) or redirects to login with the destination preserved
- On arrival, the relevant record or action is visible without additional navigation

**Success state:** User sees and can act on the context described in the notification.

**Example:** A user receives "Sarah commented on your invoice #2045." Clicking the link takes them directly to invoice #2045 with the comment visible — not to the invoices list, not to the dashboard.

This linking rule eliminates the frustration of a notification that requires the user to search for the thing the notification was about.

## Related Documents

- [Flow goals](flow-goals.md)
- [New user flow](new-user.md)
- [Dashboard layout](../layouts/dashboard.md)
- [Notifications](../interaction/notifications.md)
- [Auth pages](../layouts/auth-pages.md)
