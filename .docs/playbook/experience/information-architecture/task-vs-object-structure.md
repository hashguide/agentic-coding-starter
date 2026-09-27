---
kind: playbook
domain: experience
topics: [task-oriented, object-oriented, navigation, information-architecture, experience, information, architecture, task, vs, object, structure]
authority: guidance
---
# Task-Oriented vs Object-Oriented Structure

## Object-oriented structure
Organize around durable entities such as Projects, Customers, Orders, Automations, Repositories, or Reports.

Use when users repeatedly return to the same objects, objects have meaningful detail/history, and many actions are contextual to one object.

Strengths: stable mental model, scalable navigation, direct links, clear ownership.

Watch for: forcing every action into a database-shaped object hierarchy.

## Task-oriented structure
Organize around goals such as Review, Publish, Reconcile, Import, or Monitor.

Use when the product is a guided workflow, tasks span several entities, or users care more about completing a job than managing records.

Strengths: clear next action, lower conceptual burden, good onboarding.

Watch for: duplicate entry points when the same data appears in several workflows.

## Hybrid structure
Use persistent object areas for durable work and task surfaces for cross-object workflows. This is common in mature productivity products.

Record which model dominates each top-level area and why.
