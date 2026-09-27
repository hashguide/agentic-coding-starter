---
kind: playbook
domain: architecture
topics: [architecture, system, shape, monolith]
authority: guidance
---

# Monolith-First Architecture

A monolith is a single deployable unit containing all application code. It connects to one database, runs as one process, and deploys as one artifact. Despite its reputation as the "legacy" architecture, it is the right starting point for the vast majority of SaaS products.

## The Case for Starting Monolithic

**Faster iteration:** In a monolith, adding a feature that touches three domains means editing three files in one codebase. In microservices, it means coordinating changes across three services, three CI/CD pipelines, and three deployment schedules.

**Simpler deployment:** One artifact to build, one deployment to configure, one rollback to execute. There is no partial deployment state where some services are at version N and others are at version N-1.

**Simpler debugging:** A request's entire execution path is in one process's log output. Stack traces are complete. Profiling covers the whole request. Distributed tracing is not required.

**Shared transactions:** Business operations that span domains (creating an invoice that notifies the client and updates the user's invoice count) execute atomically in a single database transaction. No SAGA pattern required.

**The time saved compounds.** Every week you don't spend debugging cross-service issues, managing deployment coordination, and configuring service discovery is a week of feature development.

## Concrete, Specific Signals That Justify Splitting

Not: "The codebase is getting big" or "it might scale better someday."

Yes:
- **A specific component's scaling needs diverge sharply from the rest.** Video encoding requires 16 cores per job but the web server needs 2 cores to handle thousands of concurrent requests. Scaling the whole monolith to accommodate the video encoder costs 8x what it should.
- **A component has an independent deployment cadence.** The ML inference component deploys new models several times per day; the billing and user management system deploys weekly. The shared deployment pipeline is the bottleneck.
- **A component needs a different tech stack.** The main app is TypeScript/Node.js; a computationally intensive component needs Python/NumPy. Polyglot in a monolith is painful; in a service it's clean.

## Related Documents

- [Architecture overview](../../overview.md)
- [Modular monolith](modular-monolith.md)
- [Microservices overview](../microservices/overview.md)
