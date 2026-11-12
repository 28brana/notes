# 07 · Frontend → Backend → Database Communication

> **Subject:** Computer Networks (for full stack developers)
> **Status:** ✅ done
> **Last reviewed:** 2026-11-13

[← Back to Computer Networks](README.md) · [Questions for this topic →](questions/07-frontend-backend-database-qa.md)

## In one line

A user click travels through many network hops — CDN, load balancer, API, cache, database, queues. Every hop can be slow or fail, so a senior developer designs for **limited connections, timeouts, retries and partial failures**.

---

## 1. The full request path

> 🎤 **Say it like this:** "The browser calls the API through a CDN and a load balancer. The API server checks Redis first, and on a miss queries the database through a connection pool. Slow or non-urgent work like emails is pushed to a queue and handled by background workers. Each arrow is a network call, so each one needs a timeout and error handling."

```
                     static files
Browser ──HTTPS──▶  CDN  ───────────▶ (S3 / origin)
   │
   │ /api/*
   ▼
Load Balancer ──▶ API server (Node/Express) ──┬──▶ Redis (cache / sessions / rate limit)
                     (many instances)         ├──▶ Database (Postgres/MySQL via pool)
                                              ├──▶ Other services (REST / gRPC)
                                              └──▶ Queue / Kafka ──▶ Workers ──▶ email, reports…
```

Network details of each box → [08 · Production networking](08-production-networking.md)

---

## 2. Frontend → API → Backend → Database

- **Frontend never talks to the DB directly.** It calls the API; the API validates, authorizes, then queries the DB. DB credentials stay on the server.
- The DB usually lives in a **private subnet** — only the API servers can reach it.
- Each layer has its own timeout, and they should get **shorter as you go deeper**:

```
✅ Good:  Client 30s  >  API handler 25s  >  DB query 5s
❌ Bad:   Client 10s  <  API handler 60s   (client gives up, server keeps working)
```

If the inner timeout is longer than the outer, the client gives up while the server is still working (wasted work, confusing 504s).

---

## 3. Backend → Redis

> 🎤 **Say it like this:** "Redis is an in-memory key-value store, so reads are sub-millisecond. I use it for caching DB results, storing sessions, rate-limit counters, distributed locks and pub/sub. It's still a network call, so I reuse one client connection, set timeouts, and make the app degrade gracefully if Redis is down."

| Use | Example |
|---|---|
| Cache | `user:42` → JSON, TTL 5 min |
| Sessions | `sess:abc` → `{userId: 42}` |
| Rate limiting | `INCR rl:ip:1.2.3.4` with `EXPIRE 60` |
| Pub/Sub | Broadcast chat message to all WebSocket servers |
| Locks / queues | Distributed lock, BullMQ job queue |

---

## 4. Connection pooling

> 🎤 **Say it like this:** "Opening a DB connection is expensive — TCP handshake, TLS, authentication, and memory on the DB side. A connection pool keeps a set of open connections and lends them to queries, then takes them back. This makes queries faster and also caps how many connections my app opens, so I don't overload the database."

```
Request 1 ─┐                 ┌── conn 1 ──┐
Request 2 ─┼─▶  POOL (max 10)├── conn 2 ──┼──▶  PostgreSQL
Request 3 ─┘   waiting queue └── …      ──┘
```

```js
const pool = new Pool({
  max: 10,                       // max open connections per instance
  idleTimeoutMillis: 30000,      // close idle ones
  connectionTimeoutMillis: 2000, // fail fast if pool is exhausted
});
```

**Things to know:**
- Create the pool **once** at startup, not per request.
- Always release the connection (use `pool.query`, or `try/finally client.release()`), otherwise the pool leaks and every request hangs.
- Pool size is not "bigger is better" — a DB with 8 cores can only really run a few queries in parallel. Start small (10–20) and measure.

---

## 5. Database connection limits

Every DB has a max (Postgres default `max_connections = 100`). Each connection uses memory on the DB server.

```
Total connections = instances × pool size
20 pods × pool 10 = 200  → "FATAL: too many connections for role" 💥
```

**Serverless problem:** each Lambda instance opens its own connections; 500 concurrent Lambdas = 500 connections.

**Fixes:**
- Lower pool size per instance; size it as `max_connections / instances` with headroom.
- Use a **connection pooler / proxy** in between: **PgBouncer** (Postgres), **RDS Proxy** (AWS) — many app connections share a few real DB connections.
- Read replicas for read-heavy load.

---

## 6. Query timeouts

> 🎤 **Say it like this:** "A slow query holds a connection, and if many pile up the pool runs out and the whole API stops. So I set a query timeout — in Postgres `statement_timeout` — so bad queries get killed instead of blocking everything. I also set a pool acquire timeout so requests fail fast instead of waiting forever."

```sql
SET statement_timeout = '5s';   -- per session/role, or set in connection config
```

| Timeout | Protects against |
|---|---|
| Connect timeout | DB unreachable |
| Pool acquire timeout | Pool exhausted |
| Statement / query timeout | Slow query (missing index, lock wait) |
| `idle_in_transaction_session_timeout` | Code that opens a transaction and forgets it |

---

## 7. Transactions and network failures

> 🎤 **Say it like this:** "A transaction makes multiple writes all-or-nothing. If the connection drops before COMMIT, the database rolls back automatically, so data is safe. The tricky case is when the connection drops *after* I sent COMMIT but before I got the reply — I don't know if it committed. Blindly retrying could charge a customer twice, so the operation must be idempotent — for example with a unique idempotency key or unique constraint."

```
BEGIN
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT
```

| Network drops… | What happened? | Safe action |
|---|---|---|
| Before COMMIT sent | DB rolls back | Retry is safe |
| After COMMIT, before reply | **Unknown** — maybe committed | Retry only if idempotent, or check status first |

Tips: keep transactions short, never call external APIs (payment, email) inside a DB transaction, use idempotency keys ([05 · API patterns](05-api-communication-patterns.md)).

---

## 8. Cache communication and strategies

> 🎤 **Say it like this:** "The most common pattern is cache-aside: read from cache, on a miss read from DB and store in cache with a TTL. On writes I update the DB and delete the cache key. The hard parts are invalidation — stale data — and cache stampede, where a hot key expires and thousands of requests hit the DB at once."

**Cache-aside (lazy loading):**
```js
async function getUser(id) {
  const cached = await redis.get(`user:${id}`);
  if (cached) return JSON.parse(cached);              // hit
  const user = await db.findUser(id);                  // miss
  await redis.set(`user:${id}`, JSON.stringify(user), 'EX', 300);
  return user;
}
// on update: write DB, then redis.del(`user:${id}`)
```

| Strategy | How | Good for |
|---|---|---|
| **Cache-aside** | App reads cache → DB on miss → fill cache | Most read-heavy apps |
| **Write-through** | Write to cache and DB together | Data read right after write |
| **Write-back (write-behind)** | Write to cache, DB updated later | Very high writes; risk of data loss |
| **TTL expiry** | Key auto-expires | Simple staleness bound |

**Problems:**
- **Stale data / invalidation** — delete key on write, use TTL as a safety net.
- **Cache stampede** — hot key expires → many DB hits. Fix: lock so only one request rebuilds, random jitter on TTL, refresh before expiry.
- **Cache penetration** — requests for keys that don't exist always hit DB. Fix: cache "null" for a short time.

---

## 9. Message queues

> 🎤 **Say it like this:** "A message queue lets one service hand off work without waiting. The producer puts a message in the queue and returns immediately; consumers process it later. The consumer acknowledges after success; if it fails, the message is retried, and after N failures it goes to a dead-letter queue for inspection. This makes the system resilient to spikes and failures."

```
API (producer) ──▶ [ queue: send-email ] ──▶ Worker (consumer) ──ack──▶ removed
                          │ failed 5 times
                          ▼
                   Dead Letter Queue (DLQ) → alert / inspect
```

- **Ack:** message removed only after consumer confirms success.
- **At-least-once delivery:** a message can be delivered twice → consumers must be **idempotent**.
- **Visibility timeout (SQS):** message hidden while processing; reappears if not acked in time.
- Examples: RabbitMQ, AWS SQS, BullMQ (Redis).

---

## 10. Kafka basics

> 🎤 **Say it like this:** "Kafka is a distributed, append-only log. Producers write events to a topic, which is split into partitions. Each message gets an offset. Consumers in a consumer group split the partitions between them and track their offset, so messages are not deleted after reading — they're kept for a retention period and can be replayed. Ordering is guaranteed only within a partition, so I use a key like orderId to keep related events in order."

```
Topic: orders
  Partition 0: [0][1][2][3][4] ──▶ Consumer A  ┐
  Partition 1: [0][1][2]       ──▶ Consumer B  ├─ consumer group "billing"
  Partition 2: [0][1][2][3]    ──▶ Consumer B  ┘
                       ▲ offset = position in partition
```

| Term | Meaning |
|---|---|
| Topic | Category/stream of events (`orders`) |
| Partition | Ordered log, unit of parallelism |
| Offset | Message position; consumer commits how far it read |
| Consumer group | Consumers sharing the work; each partition → one consumer in the group |
| Key | Same key → same partition → ordered |

### Queue (RabbitMQ / SQS) vs Kafka

| | RabbitMQ / SQS | Kafka |
|---|---|---|
| Model | Task queue | Event log / stream |
| After consume | Message deleted | Kept (retention), replayable |
| Multiple readers | One consumer gets each message | Many groups read same events independently |
| Ordering | Limited (SQS FIFO exists) | Per partition |
| Throughput | Good | Very high |
| Use for | Background jobs, emails, tasks | Event streaming, analytics, event-driven microservices, logs |

---

## 11. Synchronous vs asynchronous communication

> 🎤 **Say it like this:** "Synchronous means the caller waits for the answer — a REST call. It's simple but couples services: if one is slow or down, the caller suffers. Asynchronous means fire an event or message and move on — the work happens later. It's more resilient and scalable, but gives eventual consistency and is harder to debug."

| | Synchronous | Asynchronous |
|---|---|---|
| Caller waits? | Yes | No |
| Example | `GET /users/42`, gRPC call, payment check | Queue job, Kafka event, webhook |
| Pros | Simple, immediate result | Decoupled, handles spikes, retries |
| Cons | Cascading failures, latency adds up | Eventual consistency, harder debugging |
| Use when | User needs the answer now | Email, notifications, reports, video processing |

**Example:** Place order → sync: validate & charge payment (user needs result). Async: send email, update analytics, notify warehouse.

---

## 12. Microservices communication

> 🎤 **Say it like this:** "Services talk synchronously with REST or gRPC when they need an immediate answer, and asynchronously with events through Kafka or a queue for everything else. They find each other through service discovery — DNS in Kubernetes. For transactions spanning services I use the saga pattern instead of distributed transactions, and the outbox pattern to reliably publish events. A correlation ID passed through every call lets me trace one request across all services."

- **Sync:** REST (simple, JSON) or **gRPC** (binary, HTTP/2, fast, typed — good for internal service-to-service).
- **Async:** events on Kafka / SNS+SQS / RabbitMQ.
- **Service discovery:** services don't hardcode IPs. Kubernetes DNS (`http://user-service:3000`), AWS Cloud Map, Consul.
- **Resilience:** timeouts, retries with backoff, circuit breakers → [09 · Troubleshooting](09-troubleshooting-and-performance.md)

**Saga pattern** — one business transaction across services as a chain of local transactions, each with a **compensating action**:
```
Order created → Payment charged → Stock reserved ✗ fails
                  ◀── refund payment ◀── cancel order   (compensations)
```

**Outbox pattern** — problem: save to DB *and* publish event; if one fails you're inconsistent. Fix: in the **same DB transaction** write the data and an `outbox` row; a separate process reads the outbox and publishes to Kafka.

**Distributed tracing / correlation ID:**
```
Browser → API (X-Request-Id: abc-123) → Order svc (abc-123) → Payment svc (abc-123)
```
Log the ID everywhere so you can search one request across services. Tools: OpenTelemetry, Jaeger, Datadog, AWS X-Ray.

---

## ⚡ Quick revision

- Browser → CDN → LB → API → Redis / DB / queue. Every arrow = timeout + error handling.
- Timeouts get shorter going deeper (client > API > DB).
- Pool connections; create once, always release, keep size small.
- Total DB connections = instances × pool size. Use PgBouncer / RDS Proxy for serverless.
- `statement_timeout` kills slow queries before they exhaust the pool.
- Drop after COMMIT = unknown result → idempotency.
- Cache-aside + TTL + delete on write. Watch for stampede.
- Queues: ack, retry, DLQ, at-least-once → idempotent consumers.
- Kafka: topic → partitions → offsets; consumer groups; order only per partition; replayable.
- Sync for "need answer now", async for everything else.
- Microservices: REST/gRPC + events, service discovery, saga, outbox, correlation ID.
