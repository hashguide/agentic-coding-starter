---
kind: playbook
domain: experience
topics: [information-architecture, review]
authority: guidance
---
# Information Architecture Review

Review the product structure from the user's mental model and tasks rather than the implementation model.

## Map the experience
List the primary user objects, jobs, top-level work areas, navigation, routes, contextual actions, search/discovery paths, settings/administration, and transitions between related objects.

## Critique questions
- Can a new user predict where a primary task lives?
- Are related tasks and information colocated?
- Are top-level navigation items genuinely peer concepts?
- Are tabs used for related views of one context rather than unrelated destinations?
- Does the structure expose database/API boundaries that users do not care about?
- Are important actions available in context rather than requiring navigation memory?
- Is hierarchy shallow enough for frequent work but deep enough to avoid clutter?
- Are advanced or infrequent controls progressively disclosed?
- Can users recover orientation after deep links, refresh, or return visits?
- Do search, recent items, command surfaces, or shortcuts help when navigation alone is insufficient?
- Does mobile preserve priority and relationships rather than merely stacking desktop UI?

## Validate
Walk the primary journeys using realistic objects and permissions. Test first-use/empty conditions, power-user repeated tasks, cross-linked objects, settings, and small screens. Compare with current category expectations when useful.

## Output
Record structural changes in `project/information-architecture.md` and feature-level placement in the relevant `experience.md`. Create validation findings for implementation mismatches or unresolved structural friction.
