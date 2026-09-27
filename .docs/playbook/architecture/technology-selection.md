---
kind: playbook
domain: architecture
topics: [technology-selection, build-vs-buy, dependencies]
authority: guidance
---
# Technology Selection

Select technology from product requirements and operational constraints, then record consequential choices.

## Evaluate
- product behavior and delivery surface
- team/agent familiarity and ecosystem fit
- security and maintenance posture
- maturity, support, release cadence, and documentation
- performance/scale requirements
- deployment environment
- interoperability and data portability
- cost and vendor limits
- migration/reversibility
- observability and testing support

## Framework/library
Use a framework when its conventions remove recurring product complexity and fit the deployment model. Use a focused library when only one capability is needed and broader framework constraints add little value.

## Managed service vs self-hosted
Managed services fit commodity infrastructure when reduced operations outweigh recurring cost and vendor dependency. Self-hosting fits requirements for control, cost profile, locality, custom behavior, or environments where the operational burden is acceptable.

## Build vs buy
Build when the capability is core differentiation, requires unusual control, or existing services create unacceptable constraints. Buy/use established infrastructure for solved specialist problems when it reduces security and operational risk.

## Dependency decision
Before adding a dependency, establish whether the platform already provides the capability, whether the dependency is actively maintained, its security/supply-chain impact, API stability, bundle/runtime cost, and exit path.

## Research
For decisions affected by versions, pricing, quotas, support status, security advisories, or current platform capabilities, verify current primary sources.

## Record
Capture the requirement, considered approaches, chosen option, decisive reasons, rejected alternatives, operational consequences, and migration triggers in project architecture or an ADR.
