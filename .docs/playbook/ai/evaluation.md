---
kind: playbook
domain: ai
topics: [evaluation, quality, regression]
authority: guidance
---
# AI Evaluation

Define AI quality in terms of the product task rather than model preference.

## Establish the evaluation target
Specify expected behavior, unacceptable behavior, output contract, latency/cost constraints, and which failures require human confirmation or fallback.

## Evaluation approaches
- deterministic/unit checks for schema, tool calls, permissions, citations, formatting, and invariants
- curated example sets for representative product tasks and edge cases
- rubric/model-assisted evaluation for semantic quality with calibrated human review
- human review for subjective, high-impact, or ambiguous behavior
- online product metrics for completion, correction, abandonment, escalation, latency, and cost

## Dataset design
Include normal cases, ambiguous requests, missing context, adversarial inputs, long context, tool/API failures, permission boundaries, and domain-specific failure cases. Version important eval sets so changes can be compared over time.

## Release use
Run relevant evaluations when prompts, models, retrieval, tools, schemas, or orchestration change materially. Define acceptance thresholds or blocking regressions based on product impact.

## Record
Document what is evaluated, dataset provenance, metric/rubric, baseline, acceptance rule, known limitations, and how production feedback enters future evaluation sets.
