---
kind: playbook
domain: backend
topics: [backend, runtime, concurrency]
authority: guidance
---

# Concurrency Models

Concurrency is how a server handles multiple tasks at once. The right model depends on whether the tasks are waiting (I/O-bound) or computing (CPU-bound). Choosing the wrong model for the workload either wastes resources or blocks the server under load.

## The Event Loop Model (Node.js)

Node.js runs on a single thread with an event loop. The event loop processes one task at a time, but I/O operations (database queries, network requests, file reads) are non-blocking: instead of waiting for the result, the event loop registers a callback and continues processing other tasks. When the I/O completes, the callback is queued.

```
Event loop tick:
  ┌──────────┐     ┌──────────┐     ┌──────────┐
  │ Request 1│     │ Request 2│     │ Request 3│
  │ DB query │─────│ Running  │─────│ Waiting  │
  │ (waiting)│     │ callback │     │ for DB   │
  └──────────┘     └──────────┘     └──────────┘
  ↑                              ↑
  I/O delegated to OS            OS completes I/O → callback queued
```

**Why it's sufficient for most workloads:** Web servers spend most of their time waiting — waiting for a database query, waiting for an external API, waiting for a file. The event loop handles thousands of concurrent waiting operations with a single thread because waiting requires no CPU.

**What breaks it:** CPU-bound work — image processing, PDF generation, complex calculations — runs synchronously on the event loop thread, blocking all other requests during that computation. A single CPU-intensive operation on an otherwise I/O-bound server can cause cascading latency spikes.

## When True Parallelism Is Needed

True parallelism — running code simultaneously on multiple CPU cores — is needed when:
- A single request involves CPU-bound computation that takes >10ms
- The workload has a significant proportion of CPU-intensive tasks (e.g., a media processing service)
- You need to saturate multiple cores for throughput on compute tasks

For true parallelism, see [parallelism.md](parallelism.md).

## async/await and the Event Loop

`async/await` in JavaScript is syntactic sugar for Promises, which are the event loop's mechanism for non-blocking I/O. An `await` suspends the current execution and yields control to the event loop — it does not block.

```typescript
// This does NOT block the event loop during the DB query
async function getInvoice(id: string): Promise<Invoice | null> {
  // await suspends here; event loop processes other requests while DB query runs
  const invoice = await db.invoice.findUnique({ where: { id } });
  return invoice;
}

// This DOES block the event loop for the duration of the computation
function calculateComplexReport(data: DataSet): Report {
  let result = 0;
  for (const item of data) {
    result += expensiveCalculation(item); // Runs synchronously, blocks the loop
  }
  return result;
}
```

The `calculateComplexReport` function should be offloaded to a worker thread or background job. See [parallelism.md](parallelism.md).

## Related Documents

- [Parallelism](parallelism.md)
- [Fault tolerance](fault-tolerance.md)
- [Workers overview](../../async/overview.md)
