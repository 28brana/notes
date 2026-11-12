# Q&A · Frontend → Backend → Database Communication

[← Back to Computer Networks](../README.md) · [Read the notes →](../07-frontend-backend-database.md)

> Short answers you can say out loud. 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

### 🟢 Q1. Why shouldn't the frontend talk to the database directly?
DB credentials would be exposed to everyone, and there'd be no place for validation, authorization or business logic. The API sits in between, and the DB stays in a private network.

### 🟢 Q2. What is connection pooling and why use it?
A pool keeps a set of open DB connections and reuses them across requests. Opening a connection is expensive (TCP, TLS, auth), so pooling makes queries faster and caps the number of connections hitting the DB.

### 🟡 Q3. What happens if you don't release a connection back to the pool?
The pool slowly runs out; new requests wait for a connection and eventually time out — the API looks "hung". Always use `pool.query` or `try/finally { client.release() }`.

### 🟡 Q4. How do you choose pool size?
Small is usually better — the DB can only run a few queries in parallel per CPU core. Start around 10–20 per instance, make sure `instances × pool size` stays below DB `max_connections`, then tune with metrics.

### 🔴 Q5. After scaling the API from 5 to 30 pods, you get "too many connections". Why and how to fix?
Each pod has its own pool: 30 × 10 = 300 connections > DB limit. Reduce per-pod pool size, put PgBouncer / RDS Proxy in front of the DB, or add read replicas.

### 🟡 Q6. Why is serverless (Lambda) tricky with relational databases?
Every concurrent Lambda opens its own connections, so a traffic spike can open hundreds and exhaust the DB. Use RDS Proxy / PgBouncer or a serverless-friendly DB/HTTP data API.

### 🟡 Q7. Why set query timeouts?
A slow query holds a connection; many of them exhaust the pool and take the whole API down. `statement_timeout` kills runaway queries and keeps the system responsive.

### 🟡 Q8. How should timeouts be arranged across layers?
Shorter as you go deeper: client > API > DB. If the DB timeout is longer than the API's, the client has already given up while the server keeps working.

### 🔴 Q9. The network drops right after you send COMMIT. Did the transaction succeed?
Unknown — it may or may not have committed. Before COMMIT, the DB rolls back. After, you must design for it: make the operation idempotent (idempotency key / unique constraint) or check the status before retrying.

### 🟡 Q10. Why not call a payment API inside a DB transaction?
It keeps the transaction (and its locks/connection) open for a slow network call, and the external call can't be rolled back. Commit first, then call externally, or use outbox/saga.

### 🟢 Q11. What is Redis used for in a backend?
Caching, sessions, rate-limit counters, distributed locks, pub/sub and job queues (BullMQ). It's in-memory so it's very fast.

### 🟡 Q12. Explain the cache-aside pattern.
Read cache first; on miss read DB and save to cache with a TTL. On write, update DB and delete the cache key so the next read refreshes it.

### 🟡 Q13. What is cache stampede and how to prevent it?
A hot key expires and thousands of requests hit the DB together. Fix with a lock so only one request rebuilds the cache, random jitter on TTLs, or refreshing the key before it expires.

### 🟡 Q14. Cache-aside vs write-through?
Cache-aside fills the cache lazily on reads — simple, may serve stale data until TTL/invalidation. Write-through writes to cache and DB together — cache always fresh, but every write is slower.

### 🟢 Q15. What is a message queue? Why use it?
A buffer where producers put messages and consumers process them later. It decouples services, absorbs traffic spikes, and lets slow work (emails, reports) run in the background with retries.

### 🟡 Q16. What is a dead-letter queue?
Where messages go after failing processing N times. Keeps them from blocking the main queue and lets you inspect and replay them later.

### 🟡 Q17. Why must queue consumers be idempotent?
Most queues give at-least-once delivery — a message can arrive twice (e.g. consumer crashed before ack). Processing twice must not double-charge or double-email.

### 🟡 Q18. Explain Kafka topic, partition, offset and consumer group.
Topic = stream of events. Partition = ordered log, unit of parallelism. Offset = position of a message in a partition. Consumer group = consumers sharing partitions; each partition is read by one consumer in the group.

### 🟡 Q19. How do you keep event order in Kafka?
Ordering is guaranteed only within a partition. Use a message key (e.g. `orderId`) so all events for that entity go to the same partition.

### 🟡 Q20. Kafka vs RabbitMQ/SQS?
Queues delete a message after it's consumed and are great for background jobs. Kafka keeps events for a retention period, many consumer groups can read the same events, and they can be replayed — great for event streaming and event-driven microservices.

### 🟢 Q21. Synchronous vs asynchronous communication?
Sync: caller waits for the response (REST, gRPC) — simple but couples services. Async: send a message/event and continue — resilient and scalable, but eventually consistent.

### 🟡 Q22. How do microservices communicate?
Sync via REST or gRPC when an immediate answer is needed; async via events (Kafka, SQS/SNS, RabbitMQ) otherwise. They find each other through service discovery (Kubernetes DNS, Cloud Map).

### 🔴 Q23. How do you handle a transaction across multiple services?
Use the saga pattern: a sequence of local transactions, each with a compensating action (refund, cancel) if a later step fails. Avoid distributed 2-phase commit.

### 🔴 Q24. What is the outbox pattern?
Write the business data and an "event" row to an outbox table in the same DB transaction. A separate process publishes outbox rows to Kafka. Guarantees the DB change and the event never get out of sync.

### 🟡 Q25. What is a correlation ID?
A unique ID (e.g. `X-Request-Id`) created at the edge and passed through every service call and log line, so you can trace one request end-to-end. OpenTelemetry/Jaeger automate this as distributed tracing.
