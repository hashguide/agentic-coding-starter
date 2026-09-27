# Documentation Routing

Use the smallest route that covers the current decision. Always inspect relevant project and feature truth before reusable guidance.

## Always start with
- `START.md`
- `project/brief.md`
- `project/constraints.md`
- `execution/current.md`

## Idea → product definition
Read:
- `playbook/product/product-type-selection.md`
- `playbook/product/project-brief.md`
- `playbook/product/user-personas.md`
- `playbook/product/user-journeys.md`
- `playbook/product/feature-prioritization.md`
- `playbook/product/competitive-baseline.md`
- `playbook/product/non-functional-requirements.md`
- `playbook/experience/design-thinking.md`

Write/update: `project/brief.md`, `users.md`, `scope.md`, `competitive-baseline.md`, `journeys.md`, `non-functional-requirements.md`.

## Information architecture / feature placement
Read:
- `playbook/experience/design-thinking.md`
- `playbook/experience/information-architecture/overview.md`
- `playbook/experience/information-architecture/mental-models.md`
- `playbook/experience/information-architecture/task-vs-object-structure.md`
- `playbook/experience/information-architecture/navigation-models.md`
- `playbook/experience/information-architecture/routes-tabs-drawers-modals.md`
- `playbook/experience/information-architecture/hierarchy-progressive-disclosure.md`
- relevant layout and journey docs under `playbook/experience/`

Write/update: `project/information-architecture.md` and relevant feature `experience.md`.

## Brand / positioning / visual direction
Read:
- `playbook/product/competitive-analysis.md`
- `playbook/experience/brand/*`
- `playbook/experience/content/writing-for-ui.md` where language/voice is affected
- `playbook/experience/visual/visual-direction-selection.md`
- `playbook/experience/visual/design-styles.md`
- `playbook/experience/visual/typography.md`
- `playbook/experience/visual/color.md`
- `playbook/experience/visual/components.md`
- `playbook/experience/visual/motion.md`
- `playbook/experience/visual/visual-critique.md`

Write/update: `project/brand.md`, `project/visual-direction.md`, and relevant feature experience specs.

## Visual direction / UI styling
Read:
- `playbook/experience/visual/visual-direction-selection.md`
- `playbook/experience/visual/design-styles.md`
- `playbook/experience/visual/typography.md`
- `playbook/experience/visual/color.md`
- `playbook/experience/visual/components.md`
- `playbook/experience/visual/motion.md`
- `playbook/experience/visual/visual-critique.md`

Write/update: `project/visual-direction.md` and feature experience specs.

## Interaction / forms / async behavior
Read relevant files in:
- `playbook/experience/interaction/`
- `playbook/experience/information-architecture/routes-tabs-drawers-modals.md`
- `playbook/backend/architecture/realtime-transport.md`
- `playbook/backend/async/`

Write/update the feature `experience.md` and `technical-design.md`.

## Frontend architecture
Read:
- `playbook/frontend/structure.md`
- `playbook/frontend/state-management.md`
- `playbook/frontend/rendering.md`
- `playbook/frontend/performance.md`
- relevant `playbook/frontend/app-types/*`
- relevant experience docs

Write/update: `project/system-architecture.md`, project NFRs, and feature technical design.

## Backend / API / domain architecture
Read:
- `playbook/architecture/overview.md`
- `playbook/architecture/system-shape/*`
- `playbook/backend/architecture/*`
- `playbook/backend/runtime/*`
- `playbook/architecture/patterns/*` only for a concrete design problem

Write/update: `project/system-architecture.md`, feature technical design, and ADRs for consequential decisions.

## Database / data model / search
Read:
- `playbook/data/*`
- `playbook/data/search-selection.md`
- `playbook/backend/runtime/database-optimization.md`
- security/privacy docs when data is sensitive

Write/update: `project/data-model.md` and feature technical design.

## Identity / permissions / tenancy
Read:
- `playbook/access/authentication/*`
- `playbook/access/models/*`
- `playbook/security/*`

Write/update: `project/access-model.md`, `security-and-privacy.md`, and feature permissions.

## Security / privacy / compliance
Read:
- `playbook/security/*`
- applicable `playbook/compliance/*`
- relevant access, data, integration and deployment docs

Write/update: `project/security-and-privacy.md`, relevant project compliance requirements, feature acceptance criteria.

## Background work / events / realtime
Read:
- `playbook/backend/architecture/sync-vs-async.md`
- `playbook/backend/async/*`
- `playbook/backend/architecture/realtime-transport.md`
- `playbook/experience/interaction/long-running-actions.md`
- concurrency/fault-tolerance guidance

Write/update: system architecture and feature technical/experience specs.

## Integrations
Read:
- `playbook/integrations/providers/overview.md`
- provider-specific guidance when applicable
- webhook/background-job/security docs

Write/update: `project/integrations.md` and feature technical design.

## Billing / monetization
Read:
- `playbook/business/monetization/*`
- `playbook/business/payments/*`
- relevant access/entitlement and provider docs

Write/update: `project/monetization.md`, access model, feature specs.

## Observability / analytics / operations
Read:
- `playbook/observability/*`
- `playbook/operations/*`
- `playbook/operations/admin/*`
- product analytics/metrics guidance

Write/update: `project/analytics-and-observability.md`, `delivery-and-operations.md`.


## Product analytics / measurement
Read:
- `playbook/analytics/*` as relevant
- `playbook/product/activation-and-retention.md`
- feature journey/acceptance docs

Write/update: `project/analytics-and-observability.md` and feature instrumentation requirements.

## AI-powered product features
Read:
- relevant `playbook/ai/*`
- `playbook/security/*` for trust boundaries
- async/realtime guidance for long-running work
- provider guidance only after the product-level AI behavior is defined

Write/update: feature experience/technical design, project architecture, security/privacy and observability requirements.

## Testing / code quality / delivery
Read:
- `playbook/quality/testing/*`
- `playbook/quality/code/*`
- `playbook/delivery/*`

Write/update: feature acceptance, delivery/operations, execution status.

## Marketing / launch
Read:
- `playbook/growth/marketing/*`
- `playbook/growth/launch/*`
- relevant experience/content docs

Write/update: `project/launch-plan.md`.

## Agent workflow / multi-agent work
Read:
- `agent-system/*` as relevant
- `playbook/agent-development/*` for deeper reference

## Cross-discipline product critique
Use when a concept, feature, milestone, or release needs a senior-level completeness pass.

Read:
- `MANIFEST.md`
- relevant project and feature truth
- the review document for each applicable discipline
- current competitive/category baseline when the product experience is being evaluated

Check applicable areas across product strategy, requirements, journeys, IA, interaction design, visual/brand direction, frontend, backend, data, access/security, performance, accessibility, analytics/observability, testing, reliability, delivery, support, and launch.

Write/update: project/feature truth for requirement changes and `validation/` for findings.

## Review routes
### Product completeness
`playbook/product/product-completeness-review.md`

### IA
`playbook/experience/information-architecture/ia-review.md`

### Visual UI
`playbook/experience/visual/visual-critique.md` plus relevant responsive/accessibility docs

### Security
`playbook/security/checklist.md` plus feature-specific security guidance

### Release
`playbook/growth/launch/checklist.md`, delivery, testing, observability and operations docs

## Skill-assisted routing
When the coding tool supports repository skills, use the matching workflow under `.agents/skills/` (or the tool-specific adapter) and let that workflow select the relevant route above.

When no skill matches, classify the task by the sections above and read the smallest applicable project, feature, and playbook set.
