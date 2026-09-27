---
kind: playbook
domain: product
topics: [product, user, personas]
authority: guidance
---

# User Personas

A persona is a concrete representation of a real user type — not a demographic average, but a specific person with specific goals, frustrations, and context of use. Personas prevent the "user" from becoming an abstraction that means whatever is convenient at any given moment. They anchor decisions: "Does this serve the persona's goal, or our internal preference?"

## Persona Template

```markdown
## [Persona Name]

**Role:** [Job title or life role]
**Goals:** [What they're trying to accomplish — outcomes, not features]
**Pain points:** [What currently frustrates them about existing solutions]
**Technical comfort:** [Novice / intermediate / power user — be specific about what they can and can't do]
**Context of use:** [When, where, and how they use the product — mobile during commute? Desktop at work? In a rush or deliberate?]
**Decision triggers:** [What causes them to try a new tool? What causes them to leave?]
```

## How Many Personas for an MVP

**Two to three personas is the right number for an MVP.** More than three and you will design for nobody — every decision becomes a negotiation between five competing needs with no tiebreaker.

At minimum, define:
- **Primary persona:** The user who experiences the core value. Every MVP feature serves them first.
- **Secondary persona:** A user who benefits from the product but whose needs are adjacent (often an admin, a team member, or a different role in the same workflow).

Define a third persona only if their needs differ enough to produce distinct design decisions. If the third persona's requirements are a subset of the first two, merge them.

**Do not define:** Aspirational future personas ("the enterprise buyer we'll pursue in year two"). Those personas produce scope creep, not useful constraints.

## Example Personas

```markdown
## Maya — Primary: Solo Freelance Designer

**Role:** Freelance UX designer, 3 years experience, works alone
**Goals:** Invoice clients quickly, track what's owed, get paid without chasing
**Pain points:** Current invoicing tool requires too many steps; clients complain PDFs look generic
**Technical comfort:** Comfortable with web apps; will not configure integrations or write code
**Context of use:** Desktop, end of project or milestone; occasionally mobile to check payment status
**Decision triggers:** Switches tools when setup takes more than 30 minutes or when a client complains

## Tom — Secondary: Occasional Client

**Role:** Marketing manager at a small company, approves vendor invoices
**Goals:** Understand what he's being billed for and pay quickly without creating an account
**Pain points:** Being forced to create an account to pay an invoice
**Technical comfort:** Non-technical; uses email and basic web tools
**Context of use:** Mobile, on the go, follows a link from an email
**Decision triggers:** Drops off if payment requires more than two steps
```

## Mapping Personas Forward

Personas feed directly into two downstream documents:

**Access models:** Personas map to roles. Maya is the primary account holder (owner); Tom may be an unauthenticated payer or a limited-access guest. Define which access model applies to each persona in [access-models/overview.md](../access/models/overview.md).

**UX flows:** Each persona has a distinct primary journey. Maya's journey starts at signup and ends at receiving payment. Tom's journey starts at an email link and ends at a payment confirmation. Map each persona's journey in [design/ux-flows/](../experience/journeys/flow-goals.md).

## Related Documents

- [Project brief](project-brief.md)
- [User journeys](user-journeys.md)
- [Access models overview](../access/models/overview.md)
- [UX flow goals](../experience/journeys/flow-goals.md)
