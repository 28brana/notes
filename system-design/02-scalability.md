# 02 · Scalability

> **Subject:** System Design
> **Status:** ✅ done
> **Last reviewed:** 2026-11-18

[← Back to System Design](README.md) · [Questions for this topic →](questions/02-scalability-qa.md)

## In one line

Scalability is the ability of a system to handle **more load** (more users, more data, more requests) by **adding resources**, without falling over and without a full redesign.

---

## 1. What is scalability?

> 🎤 **Say it like this:** "Scalability means when load grows, I can add resources and the system keeps performance acceptable. If traffic goes 10x and I can handle it by adding more servers, the system is scalable. If I have to rewrite the architecture, it's not."

Key point: scalability is **not** "is it fast?". It's "**does it stay fast as load grows?**"

| | Performance problem | Scalability problem |
|---|---|---|
| Symptom | Slow for **one** user | Fast for one user, slow under **heavy load** |
| Fix | Better code, query, index | Better architecture: more servers, caching, sharding |

### What is "load"? (load parameters)

Always ask *which* load is growing — the answer changes the design.

- **Requests per second (QPS)** — e.g. 5,000 req/s at peak
- **Read : write ratio** — feed is 100:1 read-heavy; logging is write-heavy
- **Data size** — 10 GB fits on one DB, 50 TB does not
- **Concurrent users / connections** — chat app with 1M open WebSockets
- **Fan-out** — one celebrity post must reach 50M followers

---

## 2. Vertical vs horizontal scaling

> 🎤 **Say it like this:** "Vertical scaling means making one machine bigger — more CPU, RAM. It's simple but has a hard limit and it's a single point of failure. Horizontal scaling means adding more machines behind a load balancer. It scales much further and gives fault tolerance, but the app must be stateless and we have to deal with distributed data. In practice I start vertical because it's cheap and simple, then go horizontal when needed."

```
Vertical (scale UP)                 Horizontal (scale OUT)

  ┌──────────┐                        ┌──────┐ ┌──────┐ ┌──────┐
  │          │                        │ App  │ │ App  │ │ App  │
  │  BIG     │                        └──────┘ └──────┘ └──────┘
  │  SERVER  │                             ▲       ▲       ▲
  │ 64 CPU   │                             └───────┼───────┘
  │ 512GB RAM│                             Load Balancer
  └──────────┘
```

| | Vertical (scale up) | Horizontal (scale out) |
|---|---|---|
| How | Bigger machine (CPU / RAM / disk) | More machines |
| Code changes | None | App must be stateless; data must be split/replicated |
| Limit | Hardware ceiling (biggest machine available) | Almost unlimited |
| Failure | Single point of failure | One machine dies, others keep serving |
| Cost | Gets very expensive at the top end | Many cheap machines; pay as you grow |
| Downtime to scale | Often needs a restart | Add/remove machines live |
| Good for | Early stage, databases, quick fix | Web/app servers, large scale |

> 💡 Real systems use **both**: horizontal for app servers, vertical (plus replicas) for the database until sharding is really needed.

---

## 3. Stateless servers — the key to horizontal scaling

> 🎤 **Say it like this:** "To scale app servers horizontally they must be stateless — no user session stored in server memory. I keep session in Redis or use a JWT, and files go to S3. Then any request can go to any server and I can add or kill servers freely."

**Stateful (bad for scaling):**
```
User logs in → Server A stores session in memory
Next request → load balancer sends to Server B → "Who are you?" ❌
```

**Stateless (good):**
```
User logs in → session saved in Redis (or a JWT given to the client)
Next request → any server → reads session from Redis / verifies JWT ✅
```

| State | Where to move it |
|---|---|
| Sessions | Redis, or JWT in the client |
| Uploaded files | Object storage (S3) |
| Cache | Shared cache (Redis), not in-process memory |
| Scheduled jobs | Queue + workers, not `setInterval` on every server |

**Sticky sessions** (load balancer always sends a user to the same server) is a quick workaround, but it causes uneven load and users lose their session when that server dies. Prefer stateless.

---

## 4. Scaling each layer

> 🎤 **Say it like this:** "Scaling isn't one trick — each layer scales differently. App servers scale horizontally behind a load balancer. Reads scale with caching and read replicas. Writes and big data scale with sharding. Static content goes to a CDN. Slow work goes to a queue."

### 4.1 Load balancer
Spreads traffic over servers, runs health checks, removes dead servers. (Details in Load Balancing note.)

### 4.2 Caching — reduce load on the DB
Keep hot data in memory (Redis). A cache hit is ~1 ms vs ~10–100 ms for a DB query. Even an 80% hit rate removes 80% of DB reads.

### 4.3 CDN — static content
Images, JS, CSS, videos served from edge servers near the user. Your servers don't even see those requests.

### 4.4 Database scaling (usually the real bottleneck)

App servers are easy to scale; the **database is the hard part** because it has state.

| Technique | What it is | Solves | Trade-off |
|---|---|---|---|
| **Indexes & query tuning** | Fix slow queries first | Performance | Writes slightly slower |
| **Vertical scaling** | Bigger DB machine | Everything, for a while | Ceiling, cost |
| **Read replicas** | Primary takes writes, replicas serve reads | Read-heavy load | **Replication lag** → stale reads |
| **Caching** | Redis in front of DB | Read-heavy load | Cache invalidation |
| **Connection pooling** | Reuse DB connections (PgBouncer) | Too many connections | Extra component |
| **Vertical partitioning** | Split tables/features into separate DBs (users DB, orders DB) | Load by feature | Cross-DB joins are hard |
| **Sharding (horizontal partitioning)** | Split rows across many DBs by a key (e.g. `user_id % N`) | Write-heavy load, huge data | Complex: cross-shard queries, rebalancing, hot shards |
| **NoSQL** | DBs built to scale out (Cassandra, DynamoDB) | Massive scale, simple access patterns | Weaker joins/transactions |

```
Read replicas:                         Sharding:

        writes                          user_id 1–1M   → Shard 1
App ──────────→ Primary                 user_id 1M–2M  → Shard 2
 │                 │ replicate          user_id 2M–3M  → Shard 3
 │  reads          ▼
 └──────────→ Replica 1, Replica 2
```

> 💡 Order to try: **optimize queries → cache → read replicas → vertical scale → partition/shard**. Sharding is the last resort because it's the most complex.

### 4.5 Async processing with queues
Don't make the user wait for slow work (emails, image resizing, reports). Put a job on a queue (Kafka / RabbitMQ / SQS) and let workers process it. Queues also **absorb traffic spikes** — workers drain the backlog at their own pace.

```
User → API → "Order placed ✅" (fast)
              │
              └→ Queue → Workers → send email, update analytics, generate invoice
```

### 4.6 Microservices
Split a big app into services so each part scales on its own (search service needs 50 servers, profile service needs 3). Trade-off: network calls, harder debugging, more ops work. Don't start here.

---

## 5. Auto-scaling and elasticity

> 🎤 **Say it like this:** "Scalability is the ability to grow; elasticity is growing and shrinking automatically with traffic. With auto-scaling, say on AWS or Kubernetes, I set rules like 'add a server when CPU is above 70%' and it removes them at night to save cost."

- **Scalability** = *can* handle more load by adding resources.
- **Elasticity** = *automatically* adds/removes resources as load changes.

Auto-scaling triggers: CPU %, memory, request count, queue length, latency.

Gotchas:
- New servers take time to boot (1–5 min) → for known spikes (sale at 12:00) **pre-scale** in advance.
- Scaling app servers does not help if the **DB** is the bottleneck — you just send more load to the DB.

---

## 6. The scale cube (3 ways to scale)

A neat model to mention in interviews:

| Axis | Meaning | Example |
|---|---|---|
| **X-axis** — clone | Run many identical copies behind a load balancer | 10 copies of the API server |
| **Y-axis** — split by function | Break into services by feature | Users service, Orders service, Payments service |
| **Z-axis** — split by data | Each server handles a subset of data | Sharding by `user_id` or by region |

---

## 7. Finding the bottleneck

> 🎤 **Say it like this:** "A system is only as scalable as its slowest part. Before scaling I measure — CPU, memory, DB query time, p99 latency — find the bottleneck, fix that, and then the next bottleneck shows up somewhere else."

Common bottlenecks:
- **Database** — slow queries, too many connections, lock contention (most common)
- **Single point of failure / single instance** — one cron server, one Redis
- **Shared state** — in-memory sessions, local file uploads
- **Network calls in a chain** — service A → B → C → D, latency adds up
- **Hot keys / hot shards** — one celebrity user gets all the traffic

What to measure:
- **Throughput** (req/s)
- **Latency percentiles** — p50, p95, **p99** (averages hide slow users)
- **Error rate**
- **Resource usage** — CPU, memory, disk I/O, DB connections

Load-test before launch (k6, JMeter, Locust) to find where it breaks.

---

## 8. Example: scaling from 1 user to millions

This story is a great interview answer for "how would you scale this app?"

| Stage | Users | Setup | Why change |
|---|---|---|---|
| 1 | ~100 | One server: app + DB together | Simple, cheap |
| 2 | ~10K | Separate DB server from app server | Scale app and DB independently |
| 3 | ~100K | Load balancer + multiple **stateless** app servers; sessions in Redis | One app server not enough; no SPOF |
| 4 | ~500K | Add **cache** (Redis) + **CDN** for static files | DB overloaded with repeat reads |
| 5 | ~1M | DB **read replicas** | Reads still too many for primary |
| 6 | ~5M | **Message queue + workers** for slow tasks; auto-scaling | Requests slow due to heavy work; spiky traffic |
| 7 | ~10M+ | **Shard** the DB; split into services; multi-region | Writes and data too big for one primary |

> Numbers are rough — the point is the **order** of changes: each step fixes the current bottleneck.

---

## 9. Trade-offs to mention

- **More machines = more complexity**: network failures, partial failures, data consistency, harder debugging.
- **Replicas & caches** → faster reads but **stale data** (eventual consistency).
- **Sharding** → huge scale but cross-shard joins/transactions become hard.
- **Cost**: scaling everything "just in case" wastes money — scale for the expected load + headroom.
- **Don't over-engineer**: a startup with 1,000 users doesn't need Kafka and 20 microservices.

---

## Quick revision

- Scalability = handle more load by adding resources, keeping performance OK.
- Performance problem = slow for one user; scalability problem = slow under load.
- Vertical = bigger machine (simple, limited, SPOF). Horizontal = more machines (unlimited, needs stateless + LB).
- Make app servers **stateless** — sessions in Redis/JWT, files in S3.
- DB is the usual bottleneck: optimize → cache → read replicas → shard.
- Use CDN for static, queues for slow work, auto-scaling for spiky traffic.
- Measure (p99, throughput) → find bottleneck → fix → repeat.

## References

- [System Design Primer — Scalability](https://github.com/donnemartin/system-design-primer#scalability-video-lecture)
- *Designing Data-Intensive Applications* — Ch. 1 (Scalability), Ch. 5–6 (Replication, Partitioning)
- *System Design Interview* — Alex Xu, Vol. 1, Ch. 1 "Scale from zero to millions of users"
