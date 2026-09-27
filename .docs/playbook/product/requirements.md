---
kind: playbook
domain: product
topics: [product, requirements]
authority: guidance
---

# Requirements

The project brief defines what to build. Requirements define what "built" means — precisely enough that a developer (or an AI agent) can know when each piece is done. A requirement with ambiguous acceptance criteria is not a requirement; it's a wish. This document shows how to translate the project brief into functional requirements with unambiguous definitions of done.

## User Story Format

Every functional requirement is expressed as a user story:

> **As a** [persona], **I want** [specific action], **so that** [concrete outcome].

The persona must match a defined persona from [user-personas.md](user-personas.md). The action must be specific enough to map to a single screen or API endpoint. The outcome must be the value, not the mechanism.

**Example:**

> As a **trial user**, I want to **upgrade my plan from within the app**, so that **I can access the export feature without contacting support**.

Not: "As a user, I want to manage my subscription." (Too vague — what action, what outcome?)

## Acceptance Criteria Template

Each user story gets acceptance criteria. Use a checklist format for implementation stories:

```
Given [starting state],
When [user takes action],
Then [observable outcome].
```

Or a bulleted checklist for smaller units:

```
- [ ] The upgrade modal appears inline on the export page, not as a page redirect
- [ ] The modal shows the current plan, the target plan, and the price difference
- [ ] Clicking "Upgrade" initiates a Stripe Checkout session
- [ ] On successful payment, the export button is immediately enabled without a page reload
- [ ] On failed payment, the user sees a non-technical error message and the modal remains open
```

Both formats are valid. Use Given/When/Then for flows with complex preconditions. Use checklists for UI and API behavior stories.

## Requirement Granularity

**Rule: one story = one mergeable unit of work.**

A story is correctly sized if a developer can implement and review it in a single pull request. If a story requires changes to more than two or three distinct areas (schema, API, UI), split it.

Too large:
> As a user, I want to be notified when my team members take action on shared projects.

Correctly split:
> 1. As a user, I want to receive an in-app notification when a team member comments on my project. [schema + API]
> 2. As a user, I want to see the unread notification count in the navbar. [UI]
> 3. As a user, I want to mark individual notifications as read. [API + UI]
> 4. As a user, I want to configure which notification types I receive by email. [settings UI + email]

## Example Requirements Document

```markdown
# Feature: Project Comments

## User Stories

### US-01: Add a comment
As a **project member**, I want to **post a comment on a project**, so that
**I can communicate async with my team without leaving the app**.

Acceptance criteria:
- [ ] Comment input is visible at the bottom of the project detail page
- [ ] Submitting an empty comment is prevented with an inline validation message
- [ ] After submit, the comment appears immediately (optimistic update)
- [ ] On submit failure, the optimistic update rolls back with an inline error
- [ ] Comment is persisted to the database and visible after page reload

### US-02: Notification on new comment
As a **project member**, I want to **receive a notification when someone
comments on my project**, so that **I know when action is needed without
polling the project page**.

Acceptance criteria:
- [ ] A notification is created for all project members except the commenter
- [ ] The notification links directly to the relevant project and comment
- [ ] The navbar bell badge increments immediately on notification creation
- [ ] The user can configure whether they also receive this notification by email
  (see settings/notifications)

## Out of Scope (this iteration)
- Threaded/nested comments
- Comment reactions
- Comment editing
- @mention notifications
```

## Related Documents

- [User personas](user-personas.md)
- [Feature prioritization](feature-prioritization.md)
- [Project brief](project-brief.md)
- [User journeys](user-journeys.md)
