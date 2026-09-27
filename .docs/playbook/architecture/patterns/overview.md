---
kind: playbook
domain: architecture
topics: [architecture, patterns]
authority: guidance
---

# Design Patterns Overview

Design patterns are solutions to recurring structural problems in software. They are not a checklist to apply everywhere, and they are not magic — a pattern applied to a problem it wasn't designed for creates more complexity than it removes. The value of knowing patterns is knowing *when* they fit, which is equally knowing when they don't.

## The Warning Against Pattern Overuse

Every pattern adds a layer of abstraction. Abstraction has a cost: indirection, more files, more concepts for a reader to hold in mind. That cost is only worth paying when the problem the pattern solves is actually present.

Signs you're applying a pattern where it doesn't fit:
- The pattern adds a class or interface where a function would do
- The "flexibility" the pattern provides is for use cases that don't exist yet (YAGNI)
- You can explain the pattern but can't explain what specific problem it's solving in this codebase

Use patterns reactively — reach for them when you recognize the structural problem, not proactively.

## The Three Categories

**Creational patterns** control how objects are created. They matter when object creation is complex enough that the creation logic itself needs to be abstracted or centralized.

- [Factory](creational/factory.md) — Create objects without specifying the exact class
- [Builder](creational/builder.md) — Construct complex objects step by step
- [Singleton](creational/singleton.md) — Ensure a class has only one instance
- [Prototype](creational/prototype.md) — Clone existing objects

**Structural patterns** deal with how classes and objects are composed into larger structures.

- [Adapter](structural/adapter.md) — Make incompatible interfaces compatible
- [Decorator](structural/decorator.md) — Add behavior without subclassing
- [Facade](structural/facade.md) — Simplify access to a complex subsystem
- [Proxy](structural/proxy.md) — Control access to an object
- [Repository](structural/repository.md) — Abstract data access

**Behavioral patterns** define how objects communicate and distribute responsibility.

- [Observer](behavioral/observer.md) — Publish/subscribe event-driven systems
- [Strategy](behavioral/strategy.md) — Swappable algorithms or providers
- [Command](behavioral/command.md) — Encapsulate actions as objects
- [Middleware](behavioral/middleware.md) — Chain of responsibility for request pipelines
- [State](behavioral/state.md) — Explicit state machine transitions

## Related Documents

- [Engineering principles](../../product/engineering-principles.md)
- [Architecture overview](../overview.md)
