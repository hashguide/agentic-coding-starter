---
kind: playbook
domain: experience
topics:
  - design-thinking
  - problem-framing
  - user-outcomes
  - concept-critique
authority: guidance
---
# Design Thinking for Product Decisions

Use this guidance when the shape of a product, feature, workflow, or interface is not already fixed by project truth.

## Start from the outcome

Establish:
- who is trying to accomplish something
- the trigger that brings them to the product
- the outcome they need
- current workarounds or alternatives
- constraints, risks, and knowledge level
- what evidence would show the design works

Frame the problem before choosing screens, components, navigation, or implementation patterns.

## Develop alternatives

For meaningful decisions, consider more than one plausible product or interaction approach. Evaluate each against:
- task completion and cognitive load
- discoverability and information hierarchy
- frequency and urgency of the task
- interruption/context loss
- system behavior and latency
- error and recovery needs
- accessibility and responsive behavior
- consistency with the rest of the product
- implementation and operational consequences

## Critique the preferred direction

Ask:
- Does the structure match the user's mental model?
- Is the primary path obvious without explanation?
- Are related tasks and information placed together?
- Is complexity shown only when it becomes useful?
- Does the interaction reflect whether work is instant, asynchronous, destructive, reversible, or collaborative?
- Are loading, empty, error, interruption, and return states coherent?
- Does the visual hierarchy communicate importance before decoration?
- Does the result fit the product's brand and category while remaining distinct?

## Validate

Use prototypes, rendered screens, realistic data, code behavior, competitive research, user feedback, or product analytics according to the stage and risk of the decision. Record durable conclusions in project or feature truth.

## Project documentation

Depending on scope, update:
- `project/journeys.md`
- `project/information-architecture.md`
- `project/brand.md`
- `project/visual-direction.md`
- relevant feature `experience.md`
- an ADR when the decision has durable cross-system consequences
