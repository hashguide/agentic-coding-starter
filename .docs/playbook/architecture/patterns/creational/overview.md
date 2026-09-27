---
kind: playbook
domain: architecture
topics: [architecture, patterns, creational]
authority: guidance
---

# Creational Patterns Overview

Creational patterns solve the problem of object creation — specifically, when the act of creating an object is complex enough that the creation logic itself needs to be abstracted. In simple cases, `new Thing()` is the right answer. Creational patterns are for the cases where "which thing" or "how to configure the thing" needs to be decided at runtime, deferred, or centralized.

## When Object Creation Becomes a Problem

Creation becomes worth abstracting when:
- The type of object to create depends on runtime conditions (Factory)
- Creating an object requires a multi-step process with interdependent configuration (Builder)
- Exactly one instance should ever exist and shared access needs to be managed (Singleton)
- Objects are expensive to create and you need copies of existing instances (Prototype)

## Patterns in This Category

- [Factory](factory.md) — Create objects without specifying the exact class; choose the implementation based on configuration or context
- [Builder](builder.md) — Construct complex objects step by step, where intermediate states may be valid or invalid
- [Singleton](singleton.md) — Ensure a single shared instance; common but frequently overused
- [Prototype](prototype.md) — Clone existing objects; less commonly needed in typical SaaS code

## Related Documents

- [Patterns overview](../overview.md)
- [Structural patterns overview](../structural/overview.md)
- [Behavioral patterns overview](../behavioral/overview.md)
