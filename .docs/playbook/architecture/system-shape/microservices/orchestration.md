---
kind: playbook
domain: architecture
topics: [architecture, system, shape, microservices, orchestration]
authority: guidance
---

# Microservices Orchestration

Orchestration is the mechanism for running, scaling, and managing multiple containerized services. The choice ranges from a simple Docker Compose file to full Kubernetes clusters, with several managed platforms in between that abstract the complexity.

## Options

### Docker Compose

A single YAML file defines all services, networks, and volumes. All services run on one machine. Limited to single-host deployments.

**Tradeoffs:**

| Dimension | Docker Compose |
|---|---|
| Operational complexity | Very low — one file, one command |
| Appropriate scale | Development, staging, very small production (single VPS) |
| Scaling | No horizontal scaling — single machine only |
| Learning curve | Low |
| Good for | Local development multi-service orchestration; small production deployments |

### Kubernetes

A cluster management system that schedules containers across multiple nodes, handles service discovery, load balancing, rolling deployments, and auto-scaling.

**Tradeoffs:**

| Dimension | Kubernetes |
|---|---|
| Operational complexity | Very high — significant ongoing maintenance burden |
| Appropriate scale | Multi-team, high-traffic products with specific scaling needs |
| Scaling | Horizontal pod autoscaling, cluster autoscaling |
| Learning curve | Very high |
| Good for | Large engineering organizations with dedicated platform teams |

### Managed Container Platforms

Services like Railway, Fly.io, Render, and Northflank handle the orchestration infrastructure and expose a simpler interface: deploy a container, configure environment variables, set scaling rules.

**Tradeoffs:**

| Platform | Key strength | Key limitation |
|---|---|---|
| **Railway** | Simplest DX, free tier | Less control over networking/regions |
| **Fly.io** | Edge deployment, region control | More complex configuration |
| **Render** | Simple pricing, managed databases | Slower cold starts on free tier |

**Operational complexity:** Low (relative to Kubernetes) — managed platforms handle node management, load balancing, TLS, and basic autoscaling.

## Recommendation

**Docker Compose or a managed platform (Railway, Fly.io, Render) is sufficient until there is a specific, demonstrated need for Kubernetes.**

Kubernetes is appropriate when:
- Your platform team has Kubernetes expertise and dedicated headcount to maintain it
- You have 50+ services or specific autoscaling requirements that managed platforms cannot accommodate
- You have compliance requirements that necessitate self-hosted infrastructure

At every other stage, a managed platform provides 90% of Kubernetes's value with 10% of its operational burden. If you eventually need Kubernetes, migrating to it from a managed platform is straightforward — the containers and workload definitions transfer cleanly.

## Related Documents

- [Microservices overview](overview.md)
- [Infrastructure hosting options](../../../delivery/infrastructure/hosting-options.md)
- [Infrastructure VPS Docker](../../../delivery/infrastructure/vps/docker.md)
