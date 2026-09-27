---
kind: playbook
domain: ai
topics: [ai, cost, latency, cache, retry, fallback, reliability]
authority: guidance
---
# AI Cost, Latency, and Reliability

Set per-feature expectations for latency, model usage and cost. Track input/output size, model/provider, duration, failures, retry count and estimated cost at a level that supports debugging without storing sensitive prompt content unnecessarily.

Use caching when requests are safely reusable, smaller/faster models for bounded subtasks, streaming when partial output improves experience, background jobs for long generation, and fallbacks when provider/model failure would otherwise block a critical workflow.

Retries must be bounded and aware of side effects. Budget controls should exist at user/tenant/system boundaries when usage can grow unpredictably.
