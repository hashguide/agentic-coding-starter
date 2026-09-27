---
kind: playbook
domain: architecture
topics: [monorepo, multirepo, repository, workspace, architecture, vs]
authority: guidance
---
# Monorepo vs Multiple Repositories

## Monorepo
Use when applications and packages change together, shared types/components are valuable, one team owns the system, and coordinated refactors should be atomic.

Consider workspace tooling, build caching, package boundaries, dependency direction and deployment independence.

## Multiple repositories
Use when components have independent ownership, release lifecycles, access controls, technology stacks, or genuinely separate product boundaries.

Consider contract versioning, cross-repository coordination, duplicated tooling and local development ergonomics.

## Hybrid
Use a monorepo within a product boundary and separate repositories for independently governed systems or public SDKs when that ownership boundary is real.

Select from ownership, release coupling, shared code, deployment independence and access requirements rather than repository size alone.
