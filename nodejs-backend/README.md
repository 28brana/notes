# Node.js Backend

[← Back to all subjects](../README.md)

How Node.js works under the hood (event loop, async, streams, memory) and how to build production backends with it (Express, auth, databases, caching, jobs, testing, deployment). Interviewers use these to check you can write fast, safe, non-blocking server code.

## Part 1 · Node.js Core
[Interview Q&A →](questions/nodejs-core-qa.md) · [Practice questions 📝](questions/nodejs-core-practice.md)

| # | Topic | In one line | Priority |
|---|---|---|---|
| 01 | [What is Node.js](01-what-is-nodejs.md) | V8 + libuv, single JS thread, non-blocking I/O and the thread pool | 🔥 High |
| 02 | [The event loop](02-event-loop.md) | Phases, nextTick vs Promises vs timers vs setImmediate, blocking the loop | 🔥 High |
| 03 | [Async patterns](03-async-patterns.md) | Callbacks → Promises → async/await, Promise.all / allSettled / race / any | 🔥 High |
| 04 | [Modules](04-modules.md) | CommonJS vs ES Modules, require cache, circular imports | 🔥 High |
| 05 | [Streams and buffers](05-streams-and-buffers.md) | Process data in chunks, backpressure, pipeline(), Buffer | 🔥 High |
| 06 | [EventEmitter](06-event-emitter.md) | In-process pub/sub, sync listeners, the error event, listener leaks | ⚡ Medium |
| 07 | [Error handling](07-error-handling.md) | Operational vs programmer errors, async errors in Express, crash and restart | 🔥 High |
| 08 | [Worker threads, cluster and child processes](08-worker-threads-and-cluster.md) | Use all cores, offload CPU work, run external programs | 🔥 High |
| 09 | [Memory and garbage collection](09-memory-and-garbage-collection.md) | V8 young/old generations, common leaks, heap snapshots | 🔥 High |
| 10 | [npm and packages](10-npm-and-packages.md) | Semver ^ vs ~, package-lock, npm ci, dependency types | ⚡ Medium |
| 11 | [Performance and profiling](11-performance-and-profiling.md) | Event loop lag, --inspect, flame graphs, clinic, common fixes | ⚡ Medium |

## Part 2 · Building Production Backends
[Interview Q&A →](questions/building-backends-qa.md) · [Practice questions 📝](questions/building-backends-practice.md)

| # | Topic | In one line | Priority |
|---|---|---|---|
| 12 | [Express basics](12-express-basics.md) | Routing, request/response, app structure | 🔥 High |
| 13 | [Middleware](13-middleware.md) | The request pipeline, order, next(), error middleware | 🔥 High |
| 14 | [REST API design](14-rest-api-design.md) | Resources, status codes, pagination, versioning | 🔥 High |
| 15 | [Validation](15-validation.md) | Validate and sanitize input at the edge with schemas | ⚡ Medium |
| 16 | [Authentication](16-authentication.md) | Password hashing, sessions vs JWT, protecting routes | 🔥 High |
| 17 | [Databases and ORMs](17-databases-and-orms.md) | Drivers vs ORMs, pooling, transactions, N+1 queries | 🔥 High |
| 18 | [Caching with Redis](18-caching-with-redis.md) | Cache-aside, TTLs, invalidation, Redis use cases | 🔥 High |
| 19 | [Background jobs](19-background-jobs.md) | Queues and workers for slow or retryable work | 🔥 High |
| 20 | [File uploads](20-file-uploads.md) | Multipart, streaming uploads, direct-to-S3 with signed URLs | ⚡ Medium |
| 21 | [Real-time with Socket.IO](21-realtime-with-socketio.md) | WebSockets, rooms, scaling across servers | ⚡ Medium |
| 22 | [Testing](22-testing.md) | Unit, integration and API tests, mocking | 🔥 High |
| 23 | [Config and logging](23-config-and-logging.md) | Env config, structured logs, log levels | ⚡ Medium |
| 24 | [Security best practices](24-security-best-practices.md) | OWASP basics, headers, injection, rate limits, secrets | 🔥 High |
| 25 | [Production deployment](25-production-deployment.md) | Docker, process managers, health checks, graceful shutdown | 🔥 High |
