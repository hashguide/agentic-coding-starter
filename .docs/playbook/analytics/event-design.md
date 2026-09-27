---
kind: playbook
domain: analytics
topics: [events, analytics, instrumentation, taxonomy, properties, event, design]
authority: guidance
---
# Analytics Event Design

Track meaningful user or system outcomes rather than every interface interaction.

A useful event has a stable name, clear semantic trigger, minimum properties, identity context, versioning strategy, and a product question it helps answer.

Prefer outcome-oriented events such as `automation_run_completed` over implementation-level events such as `button_clicked` when the outcome can be observed directly.

Define sensitive-data rules before instrumentation. Avoid sending secrets, raw message bodies, payment data, or unnecessary personal information to analytics systems.

Maintain an event catalog in project analytics documentation or generated schemas when the event set becomes substantial.
