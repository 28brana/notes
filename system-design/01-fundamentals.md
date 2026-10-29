# System Design Fundamentals

> **Subject:** System Design  
> **Status:** ✅ done  
> **Last reviewed:** 2026-10-29

[← Back to System Design](README.md)

## Overview

System design is the process of defining the architecture, components, and data flow of a system so it meets its **functional requirements** (what it does) and **non-functional requirements** (how well it does it: scale, latency, availability, cost).

## Key concepts

### Functional vs non-functional requirements

- **Functional:** features — "users can upload a photo", "users can follow others".
- **Non-functional:** qualities — latency < 200 ms, 99.99% availability, 10M daily users.

### Scalability

The ability to handle more load by adding resources.

| | Vertical scaling (scale up) | Horizontal scaling (scale out) |
|---|---|---|
| How | Bigger machine (CPU/RAM) | More machines |
| Pros | Simple, no code changes | Near-unlimited, fault tolerant |
| Cons | Hardware ceiling, single point of failure | Needs load balancing, distributed-state complexity |

### Latency vs throughput

- **Latency:** time to serve one request (ms).
- **Throughput:** requests served per unit time (req/s).
- Goal: maximize throughput with acceptable latency.

### Availability

Percentage of time the system is operational.

| Availability | Downtime / year |
|---|---|
| 99% ("two nines") | ~3.65 days |
| 99.9% | ~8.76 hours |
| 99.99% | ~52.6 minutes |
| 99.999% | ~5.26 minutes |

Achieved through **redundancy** (no single point of failure), **replication**, and **failover**.

### Reliability vs availability

- **Reliable:** works correctly (right results), even when faults happen.
- **Available:** responds. A system can be available but unreliable (returns wrong data).

### Consistency

Whether all clients see the same data at the same time.
- **Strong consistency:** reads always return the latest write.
- **Eventual consistency:** replicas converge over time; reads may be stale briefly.

## Typical building blocks

```
Client → DNS → CDN → Load Balancer → App Servers → Cache → Database
                                          ↓
                                    Message Queue → Workers
```

| Component | Purpose |
|---|---|
| DNS | Resolve domain → IP |
| CDN | Serve static content close to users |
| Load balancer | Spread traffic across servers |
| Cache (Redis/Memcached) | Fast reads for hot data |
| Database | Durable storage |
| Message queue (Kafka/RabbitMQ) | Decouple services, async work |

## How to approach a design problem

1. **Clarify requirements** — functional + non-functional; ask about scale.
2. **Estimate** — users, QPS, storage, bandwidth (back-of-envelope).
3. **High-level design** — boxes and arrows of main components.
4. **Data model & APIs** — schema, endpoints.
5. **Deep dive** — bottlenecks, scaling, caching, partitioning.
6. **Trade-offs** — explain what you chose and why.

### Back-of-envelope cheat sheet

- 1 day ≈ 86,400 s ≈ **10^5 s**
- 1M requests/day ≈ **~12 req/s**
- Peak QPS ≈ 2–3 × average QPS
- 1 KB × 1M = 1 GB

## Quick revision

- Start with requirements, never with components.
- Horizontal scaling + redundancy = scale and availability.
- Every choice is a trade-off: consistency vs availability, latency vs cost.

## Self-test questions

1. What is the difference between vertical and horizontal scaling?
   <details><summary>Answer</summary>

   Vertical adds power to one machine (limited, single point of failure); horizontal adds more machines (scales further, needs load balancing and handling distributed state).

   </details>

2. How much downtime per year does 99.99% availability allow?
   <details><summary>Answer</summary>

   About 52.6 minutes.

   </details>

## References

- [System Design Primer](https://github.com/donnemartin/system-design-primer)
- *Designing Data-Intensive Applications* — Ch. 1
