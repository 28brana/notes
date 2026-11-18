# Q&A · Node.js Backend — Building Production Backends

[← Back to Node.js Backend](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 12 · Express Basics

[Read the notes →](../12-express-basics.md)

> [!question]- 🟢 Q1. What is Express and what does it add on top of Node's http module?
> Express is a minimal web framework built on Node's `http` module. It adds routing (`app.get('/users/:id')`), a middleware pipeline, and helpers like `res.json()` and `res.status()`, plus parsed `req.params` and `req.query`. It's unopinionated, so the project structure is up to you.

> [!question]- 🟡 Q2. How do you structure a medium-size Express app?
> Routes map URLs to controllers, controllers handle `req`/`res` only, services hold business logic without knowing about HTTP, and repositories talk to the DB. I keep `app.js` (builds the app) separate from `server.js` (calls `listen`) so tests can import the app without opening a port. Services can then be reused from queue workers too.

> [!question]- 🟡 Q3. Express vs Fastify vs NestJS — when would you pick each?
> Express is the simple, huge-ecosystem default. Fastify is faster thanks to schema-based serialization and has validation and logging built in — good for high-throughput APIs. NestJS is an opinionated TypeScript framework with modules and dependency injection, which helps big teams keep large codebases consistent.

> [!question]- 🔴 Q4. You see "Cannot set headers after they are sent to the client" in logs. What's going on?
> The handler sent a response twice — for example `res.status(404).json()` without `return`, then code continued to `res.json(user)`, or both a callback and the main path responded. Fix by returning after sending and making sure each branch sends exactly one response. It can also happen when middleware sends a response and still calls `next()`.

---

## 13 · Middleware

[Read the notes →](../13-middleware.md)

> [!question]- 🟢 Q5. What is middleware in Express?
> A function with `(req, res, next)` that runs in order for every matching request. It can modify the request, end the response, or call `next()` to pass control on. Logging, body parsing, auth and validation are all middleware.

> [!question]- 🟡 Q6. In what order would you register middleware, and why?
> Security headers, CORS and rate limiting first so bad traffic is rejected cheaply, then the request logger, body parser and auth, then routes. After the routes comes a 404 handler, and the error handler goes last. Order matters because Express runs them strictly top to bottom.

> [!question]- 🟡 Q7. How does error-handling middleware work, and how do you catch errors from async handlers?
> Express identifies error middleware by its four arguments `(err, req, res, next)`; calling `next(err)` skips straight to it. In Express 4, a rejected promise in an async handler isn't caught, so I wrap handlers with a helper that does `.catch(next)` or use `express-async-errors`. Express 5 forwards rejected promises automatically.

> [!question]- 🔴 Q8. Some requests hang forever with no response and no error. What do you check?
> First, a middleware that neither calls `next()` nor sends a response. Second, an async handler in Express 4 that throws — the rejection never reaches the error handler. Third, a downstream call (DB, HTTP) with no timeout. I'd add request timing logs to find where it stops and set timeouts on outbound calls.

---

## 14 · REST API Design

[Read the notes →](../14-rest-api-design.md)

> [!question]- 🟢 Q9. What's the difference between 401 and 403?
> 401 means "not authenticated" — no token or an invalid/expired one, so log in again. 403 means "authenticated but not allowed" — we know who you are, but you can't access this resource, like editing someone else's order.

> [!question]- 🟡 Q10. Offset vs cursor pagination — trade-offs?
> Offset (`?page=3&limit=20`) is simple and lets you jump to any page, but the DB still scans skipped rows, so deep pages are slow, and inserts shift results. Cursor (`?cursor=<last id>`) uses `WHERE id < cursor`, stays fast at any depth and is stable for feeds, but you can't jump to page 50. I use cursor for feeds and big tables, offset for small admin lists.

> [!question]- 🟡 Q11. How do you version an API, and when do you bump the version?
> Most commonly in the URL (`/api/v1`) because it's explicit and easy to route; header-based versioning is cleaner but harder to test. I bump only for breaking changes like removing or renaming fields or changing types. Adding an optional field isn't breaking, so it stays in the same version.

> [!question]- 🔴 Q12. A mobile client retried POST /payments on a timeout and the user was charged twice. How do you fix it?
> POST isn't idempotent, so retries create duplicates. The client sends an `Idempotency-Key` header with each logical payment; the server stores the key with the result, and a repeat with the same key returns the saved response instead of charging again. A unique constraint on the key in the DB protects against races.

---

## 15 · Validation

[Read the notes →](../15-validation.md)

> [!question]- 🟢 Q13. Why validate on the server if the frontend already validates?
> Frontend validation is only for user experience — anyone can call the API directly with curl or Postman. The server is the only place you control, so every body, param and query must be checked there.

> [!question]- 🟡 Q14. How do you implement validation in Express, and what is mass assignment?
> I define a zod or joi schema per endpoint and run it in a middleware that returns 400 with field-level errors and replaces `req.body` with the parsed data. Mass assignment is when you pass `req.body` straight into `User.create()` and a client sneaks in fields like `role: "admin"`. Whitelisting fields with a strict schema prevents it.

> [!question]- 🔴 Q15. Where does "email already taken" belong — validation middleware or service? What status code?
> In the service, because it's a business rule that needs the database, not a shape check. The middleware only checks that it's a valid email string. The service returns a conflict, which maps to 409.

---

## 16 · Authentication

[Read the notes →](../16-authentication.md)

> [!question]- 🟢 Q16. How do you store passwords in a Node app?
> Never plain text or a fast hash like SHA-256. I use bcrypt (cost around 12) or argon2, which are slow on purpose and include a salt. I use the async API so hashing doesn't block the event loop during logins.

> [!question]- 🟡 Q17. Why use both an access token and a refresh token?
> The access token is a short-lived JWT, about 15 minutes, verified without a DB call, so it's fast. The refresh token lives for days, is stored hashed on the server and sent in an httpOnly cookie, so it can be revoked. If an access token leaks, the damage window is small; logout simply deletes the refresh token.

> [!question]- 🟡 Q18. Where should tokens be stored in the browser?
> The refresh token in an httpOnly, Secure, SameSite cookie so JavaScript (and XSS) can't read it. The access token is best kept in memory. `localStorage` is readable by any injected script, so it's the risky choice.

> [!question]- 🔴 Q19. A user changed their password but their stolen session still works for a week. What's wrong with the design and how do you fix it?
> Tokens are probably long-lived JWTs with no revocation. Fix: short access tokens plus server-side refresh tokens, and on password change delete all of the user's refresh tokens (or bump a `tokenVersion` checked on refresh). Add refresh token rotation with reuse detection so a stolen token used twice revokes the whole family.

---

## 17 · Databases and ORMs

[Read the notes →](../17-databases-and-orms.md)

> [!question]- 🟢 Q20. What is the N+1 query problem and how do you fix it?
> You load N rows with one query, then run one more query per row to fetch a relation — 1 + 50 queries for 50 orders. Fix with eager loading (`include` in Prisma/Sequelize, `populate` in Mongoose), a single `WHERE id IN (...)`, or DataLoader for GraphQL.

> [!question]- 🟡 Q21. Why connection pooling, and how do you size the pool?
> Opening a connection costs tens of milliseconds and the DB has a hard limit like 100 connections. A pool keeps a few connections open and reuses them, created once at startup. Size it so pods × processes × pool size stays below the DB limit, and add PgBouncer or RDS Proxy when you scale out.

> [!question]- 🟡 Q22. How do you do a transaction with an ORM, and what should you avoid inside it?
> In Prisma, `prisma.$transaction(async tx => {...})` — any error rolls everything back. Keep it short: no HTTP calls, emails or slow work inside, because it holds locks and a pooled connection the whole time.

> [!question]- 🔴 Q23. After scaling from 5 to 30 pods, the API starts failing with "too many connections". What happened?
> Each pod has its own pool, so 30 pods × pool of 10 = 300 connections, more than Postgres allows. Options: shrink the per-pod pool, put PgBouncer in transaction mode in front, or use RDS Proxy. Also check for leaks — connections not released after errors.

---

## 18 · Caching with Redis

[Read the notes →](../18-caching-with-redis.md)

> [!question]- 🟢 Q24. Explain the cache-aside pattern.
> The app checks Redis first; on a hit it returns the cached value. On a miss it reads the DB, stores the result in Redis with a TTL, then returns it. On writes it updates the DB and deletes the cache key so the next read reloads fresh data.

> [!question]- 🟡 Q25. Why Redis instead of an in-memory JS object?
> With several processes and pods, each in-memory cache is separate and goes stale differently, and it's lost on restart and bloats the heap. Redis is shared by all instances, supports TTLs and eviction, and survives app restarts.

> [!question]- 🟡 Q26. Why delete the key on write instead of updating it?
> Two concurrent writes can set the cache in the wrong order and leave an old value there indefinitely. Deleting is simpler: the next read loads the latest value from the DB. A TTL is the safety net if a delete is missed.

> [!question]- 🔴 Q27. A popular product's cache key expires and the DB CPU spikes to 100%. Why, and how do you prevent it?
> It's a cache stampede — thousands of requests miss at the same moment and all query the DB. Fix with a lock or single-flight so only one request rebuilds the key while others wait or get the stale value, refresh hot keys before they expire, or add random jitter to TTLs.

---

## 19 · Background Jobs

[Read the notes →](../19-background-jobs.md)

> [!question]- 🟢 Q28. Why move work to a background queue?
> Slow or flaky work like emails, PDFs or image processing makes the user wait and can fail the request. Enqueueing a job lets the API respond in milliseconds, and workers handle the work with retries. It also keeps CPU-heavy work off the API's event loop.

> [!question]- 🟡 Q29. How does BullMQ handle failures?
> You set `attempts` and a `backoff` (e.g. exponential from 2 s) on the job. If the handler throws, BullMQ retries; after the last attempt the job moves to the failed set, which we monitor, alert on and can replay manually.

> [!question]- 🟡 Q30. Why must job handlers be idempotent?
> Queues give at-least-once delivery — a worker can crash after doing the work but before marking the job done, so it runs again. The handler must be safe to repeat, for example checking `emailSentAt`, using a unique `jobId`, or sending an idempotency key to the payment provider.

> [!question]- 🔴 Q31. A nightly report cron is running 6 times a night. Why?
> The cron is probably a `setInterval` or `node-cron` inside the API, and there are 6 API pods, so each one fires. Fix by using a repeatable BullMQ job (one schedule, one worker picks it up), a K8s CronJob, or a distributed lock so only one instance runs it.

---

## 20 · File Uploads

[Read the notes →](../20-file-uploads.md)

> [!question]- 🟢 Q32. How do you handle file uploads in Express?
> Files arrive as `multipart/form-data`, which `express.json()` can't parse. I use multer with strict `fileSize` limits and a type filter for small files, and stream larger files to S3 instead of keeping them in memory or on local disk.

> [!question]- 🟡 Q33. What is a presigned URL and why use it for uploads?
> It's a short-lived URL signed by the server that lets the client upload directly to S3 for a specific key. The API only checks auth and signs; the bytes never pass through Node, so it saves memory, bandwidth and event loop time. Processing then happens in a worker triggered by an S3 event.

> [!question]- 🔴 Q34. Users uploading large videos make the API run out of memory. What's wrong?
> The upload is probably buffered in memory (multer memoryStorage or `req.on('data')` concatenation) with no size limit. Switch to presigned URLs for direct-to-S3 upload, or at least stream with backpressure to S3, and enforce size limits.

---

## 21 · Realtime with Socket.IO

[Read the notes →](../21-realtime-with-socketio.md)

> [!question]- 🟢 Q35. What does Socket.IO give you over plain WebSockets?
> Rooms and namespaces, automatic reconnection, acknowledgements, and a long-polling fallback when WebSockets are blocked. The trade-off is its own protocol, so a plain WebSocket client can't connect to it.

> [!question]- 🔴 Q36. Chat works with one server, but after adding a second server some users stop receiving messages. Why?
> Each server only knows its own sockets, so a message emitted on server 1 never reaches users connected to server 2. Add the Socket.IO Redis adapter so broadcasts go through Redis pub/sub to all servers, and enable sticky sessions on the load balancer for the polling handshake.

---

## 22 · Testing

[Read the notes →](../22-testing.md)

> [!question]- 🟢 Q37. What's the difference between unit and integration tests in a Node API?
> Unit tests check one function or service in isolation with dependencies like the repository mocked — they run in milliseconds. Integration tests run real pieces together, like an HTTP request through Express to a real database, using Supertest.

> [!question]- 🟡 Q38. How do you test Express routes with Supertest?
> Export the `app` without calling `listen()`, then `request(app).post('/users').send(body)` and assert on status and body. Supertest starts an ephemeral server internally. I run these against a real Postgres in Docker or Testcontainers.

> [!question]- 🟡 Q39. How do you manage the database in tests?
> A dedicated test DB, usually a container, with migrations applied once before the suite. Each test starts clean — truncate tables or run the test in a transaction that's rolled back — so tests don't depend on each other's data or order.

> [!question]- 🔴 Q40. All tests pass, but production breaks on a SQL constraint error. What's wrong with the test strategy?
> The DB was probably mocked everywhere, so real queries, constraints and migrations were never exercised. Add integration tests against a real database with the same migrations as prod, and keep mocks for external services like payment and email.

---

## 23 · Config and Logging

[Read the notes →](../23-config-and-logging.md)

> [!question]- 🟢 Q41. How should a Node app handle configuration and secrets?
> Read everything from environment variables in one config module and validate it at startup so the app fails fast if something's missing. Use `.env` only locally (never committed) and a secrets manager in production. The same build runs in every environment — that's the 12-factor approach.

> [!question]- 🟡 Q42. Why use pino instead of console.log?
> pino writes structured JSON with levels, timestamps and context fields, so log tools can filter and search it, and it's very fast. I add a request ID to every line, redact passwords and tokens, and log to stdout so the platform ships logs.

> [!question]- 🔴 Q43. A customer reports one failed checkout and you have 2 million log lines from 10 pods. How do you find what happened?
> Use the request ID — return it in the error response or a header, then search for it across all pods in the log tool to see every line for that request. If it's passed to downstream services too, I get the full path. Without a request ID, I'd fall back to user ID and time window.

---

## 24 · Security Best Practices

[Read the notes →](../24-security-best-practices.md)

> [!question]- 🟢 Q44. What does helmet do?
> It sets security-related HTTP headers in one line — Content-Security-Policy, Strict-Transport-Security, X-Content-Type-Options, and removes `X-Powered-By`. These reduce XSS, clickjacking and downgrade risks.

> [!question]- 🟡 Q45. Does CORS protect your API from attackers?
> No. CORS is enforced by browsers and only decides which websites' JavaScript can read your responses. Curl, scripts and servers ignore it, so you still need authentication, authorization and rate limiting.

> [!question]- 🟡 Q46. How do you prevent SQL and NoSQL injection in Node?
> Use parameterized queries or the ORM's query methods, never string concatenation of user input. For MongoDB, validate types so a field like `password` can't be an object such as `{ "$ne": null }`, and strip `$` operators. Schema validation on input covers both.

> [!question]- 🔴 Q47. One API request with a long input string makes the entire server unresponsive for seconds. What could it be?
> Likely ReDoS — a regex with catastrophic backtracking running on user input, which blocks the single event loop for every user. Could also be a large synchronous JSON parse or sync crypto. Fix with input length limits, safe regex patterns, and moving heavy CPU work to worker threads.

---

## 25 · Production Deployment

[Read the notes →](../25-production-deployment.md)

> [!question]- 🟢 Q48. How do you use all CPU cores with Node?
> Node runs your JS on one thread, so you run multiple processes. On a VM, PM2 cluster mode forks one worker per core and restarts crashed ones. On Kubernetes, run one process per container and scale replicas instead.

> [!question]- 🟡 Q49. What is graceful shutdown and how do you implement it?
> On SIGTERM, the app fails its readiness check, calls `server.close()` to stop new connections while finishing in-flight requests, then closes DB, Redis and queue workers and exits. A hard timeout force-exits if something hangs, before the orchestrator's SIGKILL.

> [!question]- 🟡 Q50. Liveness vs readiness probes?
> Liveness asks "is the process alive?" — failing it restarts the container, so it should be a cheap ping. Readiness asks "can it serve traffic right now?" — it can check the DB and returns 503 during startup or shutdown, which removes the pod from the load balancer without restarting it.

> [!question]- 🔴 Q51. Every deploy causes a few seconds of 502 errors. What do you check?
> Whether the app handles SIGTERM at all — if the Dockerfile uses `npm start`, the signal may not reach Node, so it gets killed mid-request. Then check readiness flips before shutdown, that `server.close()` drains connections, and that the rollout waits for new pods to be ready. Also align keep-alive timeouts between the load balancer and Node.
