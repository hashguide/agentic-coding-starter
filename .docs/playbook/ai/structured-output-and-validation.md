---
kind: playbook
domain: ai
topics: [ai, structured-output, schema, validation, parsing, structured, output, and]
authority: guidance
---
# Structured AI Output and Validation

Use structured output when model results drive program logic, persistence, tools, workflows or UI state.

Define a schema, validate every response at the boundary, reject or repair invalid structures with bounded retries, and separate model-generated content from trusted application state.

Validate semantic constraints that schemas cannot express: identifiers exist, permissions allow the action, totals are consistent, references belong to the current tenant, and enumerated choices are valid for the current workflow.
