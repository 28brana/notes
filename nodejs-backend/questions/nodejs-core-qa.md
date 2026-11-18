# Q&A · Node.js Backend — Node.js Core

[← Back to Node.js Backend](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 01 · What is Node.js

[Read the notes →](../01-what-is-nodejs.md)

> [!question]- 🟢 Q1. What is Node.js, and what are V8 and libuv?
> Node.js is a JavaScript runtime, not a language or framework. V8 compiles and runs the JS and manages memory; libuv provides the event loop, async I/O through the OS, and a thread pool. Core modules like `fs`, `http` and `crypto` sit on top.

> [!question]- 🟡 Q2. Node is "single-threaded" — so how does it handle 10,000 connections, and which operations use the thread pool?
> Only my JavaScript runs on one thread. Network sockets are handled by the OS's async mechanisms (epoll/kqueue), so they need no threads. libuv's thread pool, 4 threads by default, handles things the OS can't do async: `fs` operations, `dns.lookup`, async `crypto` like pbkdf2/scrypt, and `zlib`.

> [!question]- 🔴 Q3. Under load, file reads and outgoing HTTP calls get slow, but CPU is only 40%. The app hashes passwords with pbkdf2 on login. What's happening?
> Thread pool starvation. pbkdf2 runs on libuv's 4 pool threads, so file reads and `dns.lookup` (used when calling a hostname) queue behind the hashing. I'd raise `UV_THREADPOOL_SIZE` to match cores, move hashing to a worker pool or separate service, and rate-limit login.

---

## 02 · The Event Loop

[Read the notes →](../02-event-loop.md)

> [!question]- 🟢 Q4. What are the phases of the Node.js event loop?
> Timers runs expired `setTimeout`/`setInterval`; pending callbacks runs some deferred system errors; poll picks up new I/O and waits if idle; check runs `setImmediate`; close callbacks runs things like socket close handlers. Between every callback, the nextTick and Promise microtask queues are drained.

> [!question]- 🟡 Q5. What does this print, and why?
> ```js
> setTimeout(() => console.log('timeout'), 0);
> setImmediate(() => console.log('immediate'));
> Promise.resolve().then(() => console.log('promise'));
> process.nextTick(() => console.log('tick'));
> console.log('sync');
> ```
> `sync`, `tick`, `promise`, then `timeout` and `immediate` in either order. Sync code first, then nextTick queue, then Promise microtasks. From the main module the timeout-vs-immediate order is not guaranteed; inside an I/O callback, setImmediate always runs first.

> [!question]- 🟡 Q6. What's the difference between process.nextTick, setImmediate and setTimeout(fn, 0)?
> `nextTick` runs right after the current operation, before Promises and before the loop continues. `setImmediate` runs in the check phase, right after poll. `setTimeout(fn, 0)` runs in the timers phase after at least ~1 ms. Recursive nextTick can starve I/O, so prefer setImmediate for deferring work.

> [!question]- 🔴 Q7. API latency spikes to 2 seconds for everyone whenever one admin downloads a report. What do you check?
> Something is blocking the event loop — likely building the report with a big loop, a sync call, or `JSON.stringify` of a huge array. I'd confirm with event loop delay metrics and a CPU profile. Fix by streaming the report, paginating, or moving it to a worker thread or background job.

---

## 03 · Async Patterns

[Read the notes →](../03-async-patterns.md)

> [!question]- 🟢 Q8. What is callback hell and how did Node move past it?
> Nested error-first callbacks where each step depends on the previous one, with error checks at every level. Promises flattened this into chains with one `.catch`, and async/await made it read like sync code with `try/catch`. `util.promisify` and `fs/promises` convert older APIs.

> [!question]- 🟡 Q9. Explain Promise.all, allSettled, race and any.
> `all` resolves with every result but rejects on the first failure. `allSettled` waits for all and never rejects — you get status per item. `race` settles with the first promise to finish, success or failure — handy for timeouts. `any` resolves with the first success and rejects with `AggregateError` only if all fail.

> [!question]- 🟡 Q10. Why is using await inside forEach a bug?
> `forEach` ignores the promise each callback returns, so the loop finishes immediately and errors become unhandled. Use `for...of` with await for sequential work, or `Promise.all(items.map(...))` for parallel.

> [!question]- 🔴 Q11. An endpoint makes 3 independent DB calls of ~100 ms each and takes 300 ms. Then someone "fixes" a batch job with Promise.all over 20,000 IDs and the DB falls over. Explain both.
> The endpoint awaits sequentially; `Promise.all` brings it to ~100 ms. The batch job has the opposite problem: unbounded parallelism opens 20,000 queries at once and exhausts the connection pool. Use batches or a concurrency limiter like `p-limit` with ~10 at a time.

---

## 04 · Modules

[Read the notes →](../04-modules.md)

> [!question]- 🟢 Q12. What are the main differences between CommonJS and ES Modules?
> CommonJS uses `require`/`module.exports`, loads synchronously at runtime and allows dynamic requires. ESM uses `import`/`export`, is parsed statically before running so it supports tree-shaking, has live bindings and top-level await, and lacks `__dirname`. Node picks by `.cjs`/`.mjs` or the `"type"` field in package.json.

> [!question]- 🟡 Q13. Why does requiring the same module twice give you the same object?
> `require` caches each module after its first load, keyed by the resolved file path. Later requires return the cached `module.exports`. That's why a module that creates a DB pool acts as a singleton across the app.

> [!question]- 🔴 Q14. After a refactor, a function imported from another file is suddenly undefined at runtime, but it's clearly exported. What could cause it?
> Most likely a circular dependency: A requires B, B requires A, and B receives A's partially-filled exports. Other options: `exports = {...}` instead of `module.exports`, or mixing default and named exports between CJS and ESM. Fix the cycle by extracting shared code into a third module.

---

## 05 · Streams and Buffers

[Read the notes →](../05-streams-and-buffers.md)

> [!question]- 🟢 Q15. What are streams and what are the four types?
> Streams process data in chunks instead of loading everything into memory. Readable is a source (file read, HTTP request), Writable a destination (HTTP response), Duplex both (TCP socket), and Transform modifies data passing through (gzip).

> [!question]- 🟡 Q16. What is backpressure and how does Node handle it?
> When the consumer is slower than the producer, data piles up in memory. `write()` returns false once the buffer passes highWaterMark; the producer should pause and resume on `'drain'`. `pipe` and `pipeline` do this automatically.

> [!question]- 🟡 Q17. Why prefer stream.pipeline over .pipe, and what is a Buffer?
> `.pipe` handles backpressure but not errors — if one stream fails, the others aren't closed and can leak. `pipeline` propagates errors and destroys all streams, and has a promise version. A Buffer is a fixed-size chunk of raw bytes stored outside the V8 heap, used for files, sockets and binary data.

> [!question]- 🔴 Q18. Your CSV export endpoint crashes the server with out-of-memory when several users export at once. How do you fix it?
> It's probably loading all rows and building the whole file in memory. I'd stream rows from a DB cursor through a CSV transform into the response with `pipeline`, so memory stays flat and backpressure respects slow clients. For very big exports, generate the file in a background job and email a download link.

---

## 06 · EventEmitter

[Read the notes →](../06-event-emitter.md)

> [!question]- 🟢 Q19. What is EventEmitter, and are listeners called synchronously?
> It's Node's built-in pub/sub class: `on` registers listeners, `emit` fires a named event. Yes — `emit` calls each listener synchronously in registration order and returns after they all run. Streams, HTTP servers and `process` are all emitters.

> [!question]- 🟡 Q20. What happens if you emit 'error' with no listener, and what does MaxListenersExceededWarning mean?
> Emitting `'error'` without a listener throws, which usually crashes the process — so always attach error handlers. The warning appears when one event has more than 10 listeners, usually because a listener is added per request and never removed — a memory leak.

---

## 07 · Error Handling

[Read the notes →](../07-error-handling.md)

> [!question]- 🟢 Q21. What's the difference between operational and programmer errors?
> Operational errors are expected runtime failures — DB timeout, invalid input, 404 — and we handle them. Programmer errors are bugs, like reading a property of undefined. For bugs, the safe move is to log and let the process restart, because its state may be corrupted.

> [!question]- 🟡 Q22. How do you handle errors from async route handlers in Express?
> In Express 4, a rejected promise from an async handler isn't caught, so the request hangs. Wrap handlers so `.catch(next)` passes errors on, or use Express 5 which does this automatically. All errors go to one central error middleware with four arguments that maps them to status codes.

> [!question]- 🟡 Q23. What should you do in unhandledRejection and uncaughtException handlers?
> Log the error with full context and exit, ideally after a graceful shutdown. Since Node 15 unhandled rejections crash the process by default. Using these handlers to keep running after a bug is dangerous because the app may be in a broken state.

> [!question]- 🔴 Q24. Some requests hang forever with no error in the logs. What do you suspect?
> A swallowed or unhandled async error: an async Express 4 handler without a wrapper, a promise without `await` or `.catch`, a callback never called on one code path, or a stream without an `'error'` listener. I'd add the async wrapper and central error middleware, set request timeouts, and enable lint rules like no-floating-promises.

---

## 08 · Worker Threads, Cluster and Child Processes

[Read the notes →](../08-worker-threads-and-cluster.md)

> [!question]- 🟢 Q25. How do you use all CPU cores with Node?
> Run one process per core — with the cluster module, PM2 in cluster mode, or in Kubernetes by running more single-process pods. The processes share the port but not memory, so sessions and caches must live in something shared like Redis.

> [!question]- 🟡 Q26. Compare cluster, worker_threads and child_process.
> Cluster forks whole processes to spread HTTP load across cores. Worker threads run inside one process with their own event loop and can share memory — best for CPU-heavy JS. child_process starts external programs like ffmpeg or a Python script, via spawn, exec or fork.

> [!question]- 🟡 Q27. Should you use worker threads to speed up DB queries or HTTP calls?
> No. I/O is already non-blocking in Node, so workers add overhead without benefit. Worker threads help only with CPU-bound work like hashing, parsing or image processing, and should come from a pool rather than one per request.

> [!question]- 🔴 Q28. After moving to PM2 cluster mode with 4 instances, users get randomly logged out. Why?
> Sessions are stored in process memory, and each request may hit a different instance that doesn't know the session. Fix by storing sessions in Redis or using stateless tokens like JWT. Same goes for any in-memory cache, rate limiter or WebSocket state.

---

## 09 · Memory and Garbage Collection

[Read the notes →](../09-memory-and-garbage-collection.md)

> [!question]- 🟢 Q29. How does garbage collection work in V8, at a high level?
> V8 frees objects that are no longer reachable from roots like globals and the stack. The heap is split into a small young generation, collected very often because most objects die young, and an old generation for objects that survive, collected less often with mark-sweep-compact.

> [!question]- 🟡 Q30. What are common causes of memory leaks in Node?
> Unbounded in-memory caches or maps, event listeners added and never removed, `setInterval` timers never cleared, closures holding large objects, and globals that collect data per request. GC can't help because these are still reachable.

> [!question]- 🟡 Q31. What does --max-old-space-size do, and when do you set it?
> It sets the old-generation heap limit in MB. In containers I set it to about 75% of the memory limit so V8 collects garbage before the container gets OOM-killed. It doesn't fix leaks — it only changes when you crash.

> [!question]- 🔴 Q32. A Node service's memory grows steadily for hours and then it crashes with "heap out of memory". How do you investigate?
> First confirm heapUsed grows across GCs under steady traffic. Then run with `--inspect`, take heap snapshots at intervals, and compare which object types keep growing and what retains them. Usually it's an unbounded cache, leaked listeners or timers. Meanwhile, restarts and alerts on memory keep production stable.

---

## 10 · npm and Packages

[Read the notes →](../10-npm-and-packages.md)

> [!question]- 🟢 Q33. What's the difference between ^1.2.3 and ~1.2.3?
> Caret allows any version with the same major — up to but not including 2.0.0. Tilde allows only patch updates within 1.2.x. For 0.x versions caret is stricter and stays within the same minor.

> [!question]- 🟡 Q34. Why commit package-lock.json, and when do you use npm ci instead of npm install?
> The lockfile records the exact version and hash of every package in the tree, so all machines install the same thing. `npm ci` installs strictly from the lock, deletes node_modules first, and fails if the lock doesn't match package.json — ideal for CI and Docker. `npm install` is for development and may update the lock.

> [!question]- 🔴 Q35. A build that passed yesterday fails today with no code changes. What happened, and how do you prevent it?
> A dependency range let a new version of a direct or transitive package install, and it broke something. Commit the lockfile, use `npm ci` in CI, and roll updates in deliberately with Dependabot or Renovate plus tests.

---

## 11 · Performance and Profiling

[Read the notes →](../11-performance-and-profiling.md)

> [!question]- 🟢 Q36. What is event loop lag and why monitor it?
> It's how late callbacks run compared to when they should — a direct measure of whether the main thread is blocked. A few ms is healthy; hundreds of ms means every request is waiting. Measure it with `perf_hooks.monitorEventLoopDelay()` and alert on it.

> [!question]- 🟡 Q37. How do you profile a slow Node.js app?
> Reproduce with a load tool like autocannon or k6, then capture a CPU profile with `--inspect` in Chrome DevTools, `--cpu-prof`, or clinic flame. In the flame graph the widest bars show where CPU time goes. For memory issues, use heap snapshots.

> [!question]- 🔴 Q38. After a release, p99 latency went from 80 ms to 900 ms but p50 barely moved. Where do you look?
> A tail problem: some requests are slow or occasionally block the loop. I'd check event loop lag and GC pauses, compare the release diff for sync calls, heavy regex or big JSON, and check slow-query logs for N+1 queries or a missing index. Then profile under load to confirm the hot spot before fixing.
