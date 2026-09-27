---
kind: playbook
domain: backend
topics: [backend, runtime, concurrency, parallelism]
authority: guidance
---

# CPU-Bound Parallelism

When work is CPU-bound — heavy computation, image processing, video encoding, PDF generation — the Node.js event loop model breaks down. CPU-bound work runs synchronously on the event loop thread, blocking every other request for its duration. The solution is to offload CPU-bound work to a separate thread or process that runs in parallel.

## When CPU-Bound Work Blocks the Event Loop

```typescript
// THIS BLOCKS THE EVENT LOOP
app.post('/api/reports/generate', requireAuth, async (req, res) => {
  // generateReport is CPU-bound: it runs synchronously for 2-3 seconds
  // During this time, no other request is processed
  const report = generateReport(req.body.filters, largeDataSet);
  res.json(report);
});
```

Under load, a single 2-second CPU-bound operation on a single-threaded server serializes all requests. Ten concurrent users each trigger a 2-second report = the tenth user waits 20 seconds.

## Worker Threads

Node.js Worker Threads (v12+) run JavaScript in a separate thread with its own V8 instance. Communication happens via message passing. Worker threads are appropriate for CPU-bound work that can be expressed in JavaScript/TypeScript.

```typescript
import { Worker, isMainThread, parentPort, workerData } from 'worker_threads';
import path from 'path';

// worker.ts — runs in the worker thread
if (!isMainThread) {
  const { filters, dataSet } = workerData;
  const result = generateReport(filters, dataSet); // CPU-bound work happens here
  parentPort!.postMessage(result);
}

// In the main thread
function runReportInWorker(
  filters: ReportFilters,
  dataSet: DataSet,
): Promise<Report> {
  return new Promise((resolve, reject) => {
    const worker = new Worker(path.resolve(__dirname, 'worker.js'), {
      workerData: { filters, dataSet },
    });
    worker.on('message', resolve);
    worker.on('error', reject);
    worker.on('exit', (code) => {
      if (code !== 0) reject(new Error(`Worker exited with code ${code}`));
    });
  });
}

// Route handler — non-blocking
app.post('/api/reports/generate', requireAuth, async (req, res) => {
  const report = await runReportInWorker(req.body.filters, dataSet);
  res.json(report);
});
```

**Worker pool:** Creating a new worker per request is expensive. Use a worker pool library (`piscina` is the standard choice for Node.js) to maintain a pool of reusable workers:

```typescript
import Piscina from 'piscina';

const pool = new Piscina({
  filename: path.resolve(__dirname, 'worker.js'),
  maxThreads: 4, // Match to available CPU cores
});

app.post('/api/reports/generate', requireAuth, async (req, res) => {
  const report = await pool.run({ filters: req.body.filters, dataSet });
  res.json(report);
});
```

## Background Jobs for Long-Running CPU Work

For CPU-bound operations that take longer than a few seconds, worker threads are not the right mechanism — the HTTP request will time out. Instead, enqueue the work as a background job:

1. The API endpoint immediately returns a job ID
2. A background worker (separate process) picks up the job and performs the CPU-intensive work
3. The client polls for completion or receives a notification when done

This pattern is described in [workers/overview.md](../../async/overview.md) and applied in [how-to/setup-data-export-and-deletion.md](../../../guides/setup-data-export-and-deletion.md) for the data export flow.

```typescript
// POST /api/reports/generate → returns immediately
app.post('/api/reports/generate', requireAuth, async (req, res) => {
  const job = await jobQueue.add('generateReport', {
    userId: req.userId!,
    filters: req.body.filters,
  });
  res.json({ jobId: job.id, status: 'queued' });
});

// GET /api/reports/status/:jobId → client polls
app.get('/api/reports/status/:jobId', requireAuth, async (req, res) => {
  const job = await jobQueue.getJob(req.params.jobId);
  res.json({ status: job?.returnvalue ? 'completed' : 'processing' });
});
```

## Related Documents

- [Concurrency overview](overview.md)
- [Workers overview](../../async/overview.md)
- [Workers job patterns](../../async/job-patterns.md)
