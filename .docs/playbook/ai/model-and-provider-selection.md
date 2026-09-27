---
kind: playbook
domain: ai
topics: [ai, model, provider, routing, latency, cost, and, selection]
authority: guidance
---
# AI Model and Provider Selection

Choose models from task requirements rather than one global default.

Evaluate quality on representative tasks, structured-output reliability, tool use, context size, latency, throughput, cost, data handling, regional availability, rate limits, model stability and fallback needs.

Use routing when distinct task classes benefit materially from different models. Keep provider-specific APIs behind a boundary when fallback, cost control, evaluation or future replacement is a realistic requirement.

Record model responsibilities and quality/cost targets rather than hard-coding a provider choice into product semantics.
