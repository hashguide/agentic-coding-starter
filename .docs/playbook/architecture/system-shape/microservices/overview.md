---
kind: playbook
domain: architecture
topics: [architecture, system, shape, microservices]
authority: guidance
---

# Microservices Overview

Microservices are independently deployable services, each owning its own data and communicating over the network. They are the right answer to a specific set of problems — and the wrong answer to most problems a solo builder or small team is actually facing.

## When Microservices Are Actually Justified

The decision to break out a microservice should be driven by a specific, demonstrated forcing function — not by the expectation of one.

**Independent scaling needs:** A service (e.g., video encoding, report generation) has radically different scaling characteristics than the rest of the application. Scaling the whole monolith to accommodate it is disproportionately expensive.

**Independent team ownership:** A component is owned by a separate team with a separate engineering culture, deployment cadence, and on-call rotation. Sharing a codebase creates coordination overhead that outweighs the cost of network communication.

**Independent deploy cadence:** A component must deploy multiple times per day while the rest of the application deploys weekly. The monolith's release process cannot accommodate this.

## The Real Costs That Solo Builders Underestimate

**Network reliability:** Every service-to-service call can fail. In a monolith, a function call either succeeds or throws synchronously. In microservices, every call introduces the possibility of timeout, partial failure, and network partition. Circuit breakers, retry logic, and timeout configuration all become required infrastructure.

**Distributed debugging:** A request that fails in a microservices architecture may have passed through three services before failing. Correlating logs across services, tracing the exact execution path, and identifying which service introduced a bug requires distributed tracing infrastructure (OpenTelemetry, Jaeger, Datadog APM) that a monolith doesn't need.

**Data consistency complexity:** In a monolith with one database, a transaction guarantees atomicity. In microservices with separate databases, cross-service data consistency requires eventual consistency patterns (SAGA, outbox pattern, two-phase commit) — each of which adds implementation complexity and failure modes.

**Operational overhead:** Each service needs its own CI/CD pipeline, its own monitoring and alerting, its own deployment configuration, its own logging setup, and its own on-call rotation. A monolith has one of each.

## Starting Point

Start with a monolith. Structure it as a modular monolith (see [monolith/modular-monolith.md](../monolith/modular-monolith.md)) to keep module boundaries clean. The work you invest in module boundaries today is what makes breaking out a microservice tractable later, when you have a specific, demonstrated reason to do so.

## Related Documents

- [Architecture overview](../../overview.md)
- [Monolith overview](../monolith/overview.md)
- [Modular monolith](../monolith/modular-monolith.md)
- [Service boundaries](service-boundaries.md)
