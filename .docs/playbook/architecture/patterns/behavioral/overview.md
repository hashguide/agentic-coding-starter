---
kind: playbook
domain: architecture
topics: [architecture, patterns, behavioral]
authority: guidance
---

# Behavioral Patterns Overview

Behavioral patterns define how objects communicate and distribute responsibility. They answer questions like: How do unrelated components react to the same event without knowing about each other? How do you swap algorithms at runtime? How do you prevent ad-hoc boolean flags from creating invalid state combinations?

## Why Communication Patterns Matter

As a codebase grows, the naive approach to communication — calling functions directly — creates tight coupling. When the user is created, the route handler calls sendWelcomeEmail(), trackAnalyticsEvent(), createOnboardingJob(), and sendSlackNotification() — all inline. Adding a fifth reaction requires modifying the route handler. Removing one requires knowing all its callers.

Behavioral patterns provide structure for this communication: events for decoupled reactions, strategies for swappable behavior, state machines for valid transitions, middleware for sequential processing.

## Patterns in This Category

- [Observer](observer.md) — Publish/subscribe event-driven decoupling; multiple listeners react to one event
- [Strategy](strategy.md) — Swappable algorithms or providers behind a common interface
- [Command](command.md) — Encapsulate actions as objects; enables queuing, undo, and logging
- [Middleware](middleware.md) — Chain of responsibility for processing pipelines (HTTP request handling)
- [State](state.md) — Explicit state machines that prevent invalid state transitions

## Related Documents

- [Patterns overview](../overview.md)
- [Structural patterns overview](../structural/overview.md)
- [Creational patterns overview](../creational/overview.md)
