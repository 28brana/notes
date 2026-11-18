# Q&A · What is System Design

[← Back to System Design](../README.md) · [Read the notes →](../01-what-is-system-design.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

> [!question]- 🟢 Q1. What is system design?
> Planning the architecture of a system — its components, how they talk to each other, and where data is stored — so it meets the feature requirements and also scales, stays available, and stays fast as usage grows.

> [!question]- 🟢 Q2. HLD vs LLD?
> HLD (high-level design) is the big picture: services, databases, caches, queues and how they connect. LLD (low-level design) is inside one component: classes, interfaces, DB schema, design patterns.

> [!question]- 🟢 Q3. Functional vs non-functional requirements?
> Functional = what the system does (upload photo, send message). Non-functional = how well it does it (latency, availability, scale, consistency, security). Non-functional ones drive most design decisions.

> [!question]- 🟢 Q4. Latency vs throughput?
> Latency is the time to serve one request. Throughput is how many requests are served per second. We want high throughput with acceptable latency.

> [!question]- 🟢 Q5. What does 99.99% availability mean?
> The system is up 99.99% of the time — about 52 minutes of downtime per year. Each extra "nine" cuts downtime by 10x and costs a lot more.

> [!question]- 🟡 Q6. Availability vs reliability?
> Available = the system responds. Reliable = it responds correctly even when parts fail. A system returning wrong data is available but not reliable.

> [!question]- 🟡 Q7. Strong vs eventual consistency — give examples.
> Strong: every read sees the latest write — needed for bank balances, inventory, payments. Eventual: replicas catch up after a short delay — fine for likes, view counts, feeds.

> [!question]- 🟡 Q8. What is a single point of failure (SPOF)?
> A component whose failure takes the whole system down — one DB, one load balancer, one server. Fix it with redundancy: replicas, multiple instances, failover.

> [!question]- 🟡 Q9. How do you approach a system design interview question?
> Clarify requirements and scale → back-of-envelope estimates → high-level design → APIs and data model → deep dive on bottlenecks → discuss trade-offs and failures.

> [!question]- 🟡 Q10. 10M DAU, each making 10 requests/day — what's the QPS?
> 100M requests/day ÷ ~10^5 seconds ≈ 1,000–1,200 QPS average. Peak is around 2–3x, so ~3,000 QPS.

> [!question]- 🔴 Q11. Why do you ask "read-heavy or write-heavy?" first?
> It decides the design. Read-heavy (feeds, product pages) → caching, CDN, read replicas. Write-heavy (logging, metrics, chat) → queues, write-optimized DBs (Cassandra), sharding.

> [!question]- 🔴 Q12. Why is there no "best" design?
> Every choice is a trade-off: consistency vs availability, latency vs cost, simplicity vs scale. The right design depends on requirements — so I always explain *why* I chose something and what I gave up.
