# 01 · What is System Design

> **Subject:** System Design
> **Status:** ✅ done
> **Last reviewed:** 2026-11-18

[← Back to System Design](README.md) · [Questions for this topic →](questions/01-what-is-system-design-qa.md)

## In one line

System design is deciding **which parts** a software system needs, **how they talk to each other**, and **where the data lives**, so the system does what users need *and* keeps working well when lots of people use it.

---

## 1. What is system design?

> 🎤 **Say it like this:** "System design is planning the architecture of a system before building it — the components, how data flows between them, and how it will handle growth and failures. Writing code solves one problem on one machine. System design is about making that code work for millions of users, across many machines, without going down."

Simple way to think about it:

- **Coding** = how to build one room.
- **System design** = the blueprint of the whole building: how many floors, where the stairs and lifts go, what happens in a fire, how to add more floors later.

Two levels people talk about:

| | High-Level Design (HLD) | Low-Level Design (LLD) |
|---|---|---|
| Focus | Big boxes: services, databases, caches, queues | Inside one box: classes, functions, DB schema |
| Question it answers | "What are the parts and how do they connect?" | "How exactly is this part coded?" |
| Example | App servers → Redis → Postgres, plus Kafka for events | `Order` class, `OrderRepository`, design patterns |
| Interview round | "Design Instagram / URL shortener" | "Design a parking lot / elevator system" |

> When people say "system design interview", they usually mean **HLD**.

---

## 2. Why does it matter?

> 🎤 **Say it like this:** "A design that works for 100 users can fall over at 1 million. Good system design lets us scale, stay available when things fail, keep latency low, and control cost. It's also about trade-offs — there is no perfect design, only the right one for the requirements."

Without good design you get:
- The site goes down on a sale day (can't handle traffic spikes).
- One server crash takes the whole product down (single point of failure).
- Pages get slow as data grows (no caching, no indexes, one big DB).
- Cloud bill explodes (scaling the wrong thing).

---

## 3. Functional vs non-functional requirements

> 🎤 **Say it like this:** "Functional requirements are *what* the system does — the features. Non-functional requirements are *how well* it does them — scale, latency, availability, consistency, security. In an interview I always clarify both before I draw anything."

| Functional (features) | Non-functional (qualities) |
|---|---|
| User can sign up and log in | 10M daily active users |
| User can upload a photo | Feed loads in < 200 ms |
| User can follow others | 99.99% availability |
| User sees a feed of followed people | Photos must never be lost (durability) |

Most of system design is driven by the **non-functional** side. "Post a photo" is easy; "post a photo for 500M users with < 200 ms latency" is the real problem.

---

## 4. Core concepts (the vocabulary)

These words come up in every system design discussion. Each one gets its own detailed note.

| Concept | Simple meaning | Detailed note |
|---|---|---|
| **Scalability** | Can it handle more load by adding resources? | [02 · Scalability](02-scalability.md) |
| **Availability** | Is it up and responding? (measured in "nines") | 📝 |
| **Reliability** | Does it give correct results even when parts fail? | 📝 |
| **Latency** | How long one request takes (ms) | 📝 |
| **Throughput** | How many requests per second it can handle | 📝 |
| **Consistency** | Does everyone see the same latest data? | 📝 |
| **Fault tolerance** | Keeps working when a part breaks | 📝 |
| **Durability** | Once saved, data is not lost | 📝 |
| **Maintainability** | Easy for engineers to change and operate | 📝 |

### Quick versions

**Latency vs throughput** — Latency = time for one car to cross a bridge. Throughput = how many cars cross per hour. You can have high throughput but bad latency (a wide, slow road).

**Availability in "nines"**

| Availability | Downtime / year |
|---|---|
| 99% ("two nines") | ~3.65 days |
| 99.9% | ~8.76 hours |
| 99.99% | ~52.6 minutes |
| 99.999% | ~5.26 minutes |

Achieved with **redundancy** (no single point of failure), **replication**, and **failover**.

**Reliability vs availability** — Available = it answers. Reliable = it answers *correctly*. A system that returns wrong data quickly is available but not reliable.

**Strong vs eventual consistency** — Strong: every read gets the latest write (bank balance). Eventual: replicas catch up after a short delay (like count on a post).

---

## 5. Common building blocks

> 🎤 **Say it like this:** "Most web systems are built from the same blocks: DNS, CDN, load balancer, stateless app servers, a cache, a database, and a message queue with workers for background jobs. The design is mostly about choosing and connecting these for the given requirements."

```
                  ┌──────── CDN (images, JS, CSS)
                  │
Client → DNS → Load Balancer → App Servers (stateless) → Cache (Redis) → Database
                                     │
                                     └→ Message Queue → Workers (emails, video processing)
```

| Component | What it does | Examples |
|---|---|---|
| DNS | Turns domain name into IP | Route 53, Cloudflare |
| CDN | Serves static files from a server near the user | CloudFront, Cloudflare, Akamai |
| Load balancer | Spreads traffic across servers, removes unhealthy ones | Nginx, AWS ALB, HAProxy |
| App servers | Run business logic | Node.js, Java, Go services |
| Cache | Keeps hot data in memory for fast reads | Redis, Memcached |
| Database | Durable storage | Postgres, MySQL, MongoDB, Cassandra |
| Object storage | Big files (images, videos) | S3, GCS |
| Message queue | Decouples services, runs work asynchronously | Kafka, RabbitMQ, SQS |

---

## 6. How to approach a design question (interview framework)

> 🎤 **Say it like this:** "I start by clarifying requirements and scale, do a quick estimate, draw a simple high-level design, define the APIs and data model, then go deep on the bottlenecks and explain my trade-offs."

| Step | What you do | Time (45 min interview) |
|---|---|---|
| 1. Clarify requirements | Functional + non-functional. Ask: how many users? read-heavy or write-heavy? | ~5 min |
| 2. Estimate | Users → QPS, storage, bandwidth | ~5 min |
| 3. High-level design | Boxes and arrows for the main flow | ~10 min |
| 4. APIs & data model | Endpoints, tables/collections, choose SQL vs NoSQL | ~5 min |
| 5. Deep dive | Bottlenecks: caching, sharding, queues, failures | ~15 min |
| 6. Wrap up | Trade-offs, what you'd improve, monitoring | ~5 min |

**Good questions to ask the interviewer:**
- How many users (DAU)? Expected growth?
- Read-heavy or write-heavy? (Twitter feed = read-heavy; logging = write-heavy)
- Do we need strong consistency or is eventual OK?
- Any latency target? Global users or one region?
- What's in scope and what's out of scope?

### Back-of-envelope cheat sheet

- 1 day = 86,400 s ≈ **10^5 s** (round it, interviewers are fine with that)
- 1M requests/day ≈ **~12 req/s**
- 100M requests/day ≈ **~1,200 req/s**
- Peak QPS ≈ **2–3 × average**
- 1 KB × 1M = 1 GB · 1 MB × 1M = 1 TB

**Example:** 10M DAU, each makes 10 requests/day → 100M/day → ~1,200 QPS average → ~3,000 QPS peak.

---

## 7. Mistakes to avoid in interviews

- Jumping straight into drawing Kafka and Redis before understanding the problem.
- Not asking about scale — designing for 100 users when they meant 100M (or the opposite: over-engineering).
- Staying silent — think out loud, the interviewer grades your reasoning.
- Saying "this is the best way" — always say **"the trade-off is…"**.
- Ignoring failures — what happens if the DB or a server dies?

---

## Quick revision

- System design = components + how they talk + where data lives + how it scales and fails.
- HLD = big boxes; LLD = classes and code.
- Functional = features; non-functional = scale, latency, availability, consistency.
- Always start from requirements, never from components.
- Every decision is a trade-off — say it out loud.

## References

- [System Design Primer](https://github.com/donnemartin/system-design-primer)
- *Designing Data-Intensive Applications* — Martin Kleppmann, Ch. 1
- *System Design Interview* — Alex Xu, Vol. 1, Ch. 1–3
