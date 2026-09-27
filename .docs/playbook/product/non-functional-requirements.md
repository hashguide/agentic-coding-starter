---
kind: playbook
domain: product
topics: [nfr, performance, scale, reliability, accessibility, support, constraints, product, non, functional, requirements]
authority: guidance
---
# Non-Functional Requirements

Non-functional requirements describe qualities and constraints that can change product acceptance or architecture. Define measurable targets where they affect a real decision.

## Performance
Consider interaction latency, page/render performance, API latency, job completion expectations, search latency, upload/processing time, and resource usage.

## Workload and scale
Estimate launch and plausible near-term concurrency, request/job rates, dataset size, file volume, event volume, and growth shape. Use ranges where exact numbers are unknown.

## Availability and recovery
Define availability expectations, acceptable degraded behavior, backup needs, recovery time, recovery point, and dependency failure tolerance when product impact warrants them.

## Platform support
Define supported browsers, devices, operating systems, runtimes, offline/connectivity expectations, screen sizes, installation/update constraints, and accessibility target.

## Security, privacy and compliance
Capture data classification, residency, retention, encryption, audit, regulatory, age/industry, and contractual constraints that can change system design.

## Cost and operations
Record meaningful infrastructure budgets, third-party quotas, model/API cost limits, operational staffing, support response expectations, and deployment constraints.

## Decision process
For each proposed NFR ask:
1. What user or business outcome requires it?
2. How will it be measured?
3. What design decisions does it constrain?
4. What happens if the target is missed?

Write accepted targets into `project/non-functional-requirements.md` and connect them to architecture and acceptance criteria.
