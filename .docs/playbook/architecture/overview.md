---
kind: playbook
domain: architecture
topics: [architecture]
authority: guidance
---

# Architecture Overview

The first architectural decision is the most consequential: monolith, modular monolith, or microservices. This decision affects every subsequent choice — deployment complexity, debugging tooling, data consistency strategy, and team coordination overhead. Most builders choose the wrong one in the wrong direction: they start with microservices because they've heard monoliths don't scale, or they build an unstructured monolith that becomes unmaintainable.

## The Three Options

### Monolith

A single deployable unit containing all application code. One codebase, one deploy, one database.

### Modular Monolith

A single deployable unit whose internal code is organized into modules with strict boundaries. The modules could theoretically be separated into services, but haven't been yet.

### Microservices

Multiple independently deployable services, each owning its own data store, communicating over the network.

## Comparison Table

| Dimension | Monolith | Modular Monolith | Microservices |
|---|---|---|---|
| Team size fit | Solo to small | Small to medium | Medium to large (multiple teams) |
| Iteration speed | Fast | Fast | Slow (coordination overhead) |
| Operational complexity | Low | Low | High (service discovery, distributed tracing, N network hops) |
| Deployment complexity | Simple | Simple | Complex (independent deploy pipelines per service) |
| Debugging | Easy (single process) | Easy | Hard (distributed transactions, cross-service logs) |
| Data consistency | Simple (single DB) | Simple | Hard (distributed transactions, eventual consistency) |
| Appropriate scale | Most SaaS products | Most growing SaaS products | Large-scale, multi-team products with specific scaling needs |

## Recommendation

**Start monolithic. Default to modular monolith. Only split to microservices when there is a clear, specific, demonstrated forcing function.**

A modular monolith gives you the organizational benefits of microservices (clear ownership, module boundaries, explicit interfaces) without the operational costs (network reliability, distributed debugging, data consistency complexity). The cost of moving from a modular monolith to microservices is much lower than the cost of building microservices from the start and discovering you didn't need them.

**Forcing functions that justify microservices:**
- A specific component must scale independently at a rate that would require scaling the entire monolith to an impractical degree
- A specific component has independent deployment cadence requirements that the monolith's shared deploy cannot accommodate
- A specific component is owned by a separate team with separate engineering practices that cannot coexist in one repo

"It might scale better someday" is not a forcing function.

## Related Documents

- [Monolith overview](system-shape/monolith/overview.md)
- [Modular monolith](system-shape/monolith/modular-monolith.md)
- [Microservices overview](system-shape/microservices/overview.md)
- [Backend structure](../backend/architecture/structure.md)
- [Non-functional requirements](../product/non-functional-requirements.md)
