---
kind: playbook
domain: architecture
topics: [architecture, patterns, structural]
authority: guidance
---

# Structural Patterns Overview

Structural patterns deal with how classes and objects are composed into larger structures. They answer questions like: How do I make two incompatible interfaces work together? How do I add behavior without modifying the original class? How do I simplify a complex subsystem for callers who don't need to understand all of it?

## Why Object Composition Matters

In a growing codebase, you frequently encounter situations where:
- An existing component doesn't match the interface you need (Adapter)
- You want to add cross-cutting behavior (logging, caching) without touching the original implementation (Decorator)
- A subsystem has become complex and callers shouldn't need to know all of it (Facade)
- Access to an object should be controlled or mediated (Proxy)
- Data access logic is scattered across the codebase instead of centralized (Repository)

Each structural pattern addresses one of these problems.

## Patterns in This Category

- [Adapter](adapter.md) — Wrap an incompatible interface to match what callers expect
- [Decorator](decorator.md) — Add behavior (logging, caching, validation) without subclassing
- [Facade](facade.md) — Provide a simplified interface over a complex subsystem
- [Proxy](proxy.md) — Control access, add caching, or lazy-load resources
- [Repository](repository.md) — Abstract data access into a centralized, swappable layer

## Related Documents

- [Patterns overview](../overview.md)
- [Creational patterns overview](../creational/overview.md)
- [Behavioral patterns overview](../behavioral/overview.md)
