---
kind: playbook
domain: agent-development
topics: [agent, development, task, sizing]
authority: guidance
---

# Agent Task Sizing

Agents produce better results on well-scoped tasks than on broad ones. A task that is too large produces a large diff that is hard to review, mixes unrelated concerns, and gives the agent too many opportunities to make conflicting decisions. A task that is too small is overhead without benefit. The right size is: one mergeable, working increment.

## Criteria for a Well-Sized Task

A task is correctly sized for an agent if:

1. **It produces one mergeable, working increment.** The output can be reviewed and merged without waiting for another task to make it functional.
2. **It touches one domain.** Auth, billing, notifications, and data export are separate domains. A task that touches two or more is a candidate for splitting.
3. **It has a clear definition of done.** "Build the notification system" is not a definition of done. "Create the notifications table, add the API endpoints for listing and marking notifications as read, and write tests for both endpoints" is.

## Examples of Tasks That Are Too Large

| Too large | Why | Split into |
|---|---|---|
| "Build the auth system" | Touches schema, sessions, OAuth, RBAC, middleware, UI | Schema → sessions → OAuth → RBAC → login UI → auth middleware |
| "Implement payments" | Checkout, webhooks, subscription lifecycle, dunning | Checkout session → webhook handler → subscription state machine → dunning emails |
| "Set up the admin panel" | User management, billing lookup, feature flags, audit log | Each feature in admin is its own task |

## Worked Example: Breaking Down "Build Auth"

"Build auth" is 3–5 days of work and touches every layer. Here is how to break it into agent-sized tasks:

1. **Schema:** Add `users`, `sessions`, and `roles` tables with migrations. No business logic. Definition of done: migration runs cleanly, schema matches data-model-overview.
2. **Session management:** Implement session creation and validation middleware. Input: valid credentials. Output: session cookie set. No OAuth yet.
3. **Login/signup API:** Implement the login and signup endpoints using the session middleware. Definition of done: integration tests pass for both happy path and error cases.
4. **Google OAuth:** Add the OAuth redirect and callback routes. Account linking by provider user ID (not email). Definition of done: can sign in with a real Google account in the dev environment.
5. **RBAC middleware:** Add role checking to protected routes. Definition of done: an admin route returns 403 for a non-admin user, confirmed by test.
6. **Auth UI:** Login page, signup page, error states per `design/layouts/auth-pages.md`. Definition of done: all states render correctly, error message follows auth enumeration rule.

Each task is one PR, one review session, one logical increment.

## Signs a Task Is Too Small

A task is probably too small if:
- The setup (reading context, opening files, confirming conventions) takes longer than the work
- The output is a single helper function that has no independently testable behavior
- The PR has fewer than 10 meaningful lines changed

In these cases, combine related small tasks into one.

## Related Documents

- [Context management](context-management.md)
- [Multi-agent workflows](multi-agent-workflows.md)
- [Prompt library](prompt-library.md)
