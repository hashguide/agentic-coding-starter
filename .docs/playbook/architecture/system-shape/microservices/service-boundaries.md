---
kind: playbook
domain: architecture
topics: [architecture, system, shape, microservices, service, boundaries]
authority: guidance
---

# Microservice Boundaries and Communication

When microservices are warranted, the hardest part is not the implementation — it's defining where service boundaries are and how services communicate. Wrong boundaries create services that are tightly coupled despite being separately deployed, producing all the operational costs of microservices with none of the independence benefits.

## Bounded Contexts (Domain-Driven Design Lite)

A bounded context is the area within which a specific model and its language are consistently applied. A service boundary should correspond to a bounded context — the service owns everything within that context and nothing outside it.

Identifying bounded contexts:
1. Look for areas of the domain where the same word means different things. "User" in the billing context is a Stripe customer with a payment method. "User" in the notification context is a recipient with preferences. These are different models — different bounded contexts.
2. Look for teams that naturally form around distinct areas of the domain.
3. Look for data that almost never crosses a domain boundary without transformation.

## Synchronous vs. Asynchronous Communication

| Pattern | When appropriate | Risks |
|---|---|---|
| **REST/gRPC (sync)** | When the caller needs an immediate response to proceed | The caller fails if the callee is down; latency is additive |
| **Message queue/event (async)** | When the caller can proceed without an immediate response | Eventual consistency; harder to debug; requires a message broker |

**The rule against synchronous service-to-service chains:** If Service A calls Service B which calls Service C synchronously, a failure in C fails B which fails A. The entire chain is as available as its least available service. Each hop multiplies latency (A's p99 = sum of B's p99 + C's p99 + A's own p99).

Synchronous chains longer than one hop are a design signal that services are too tightly coupled. Refactor: either consolidate the services, or convert the communication to asynchronous events.

## Communication Patterns

**Request-response (gRPC/REST):** Use for operations where the caller needs the result to proceed. Example: the API gateway calling the billing service to check if a user has an active subscription before allowing access.

**Event-driven (message queue):** Use for side effects that can be processed independently. Example: the order service publishes `order.created` — the notification service and analytics service subscribe and process independently.

```typescript
// Event-driven: order service publishes, never knows who is listening
await messageQueue.publish('order.created', {
  orderId: order.id,
  userId: order.userId,
  total: order.total,
  timestamp: new Date().toISOString(),
});

// Notification service subscribes independently
messageQueue.subscribe('order.created', async (event) => {
  await notificationService.send({
    userId: event.userId,
    message: `Order #${event.orderId} received`,
    link: `/orders/${event.orderId}`,
  });
});
```

## Related Documents

- [Microservices overview](overview.md)
- [Microservices orchestration](orchestration.md)
- [Workers overview](../../../backend/async/overview.md)
- [Observer pattern](../../patterns/behavioral/observer.md)
