# Q&A · Scalability

[← Back to System Design](../README.md) · [Read the notes →](../02-scalability.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

> [!question]- 🟢 Q1. What is scalability?
> The ability to handle more load — users, requests, data — by adding resources, while keeping performance acceptable and without redesigning the system.

> [!question]- 🟢 Q2. Vertical vs horizontal scaling?
> Vertical = bigger machine (more CPU/RAM): simple, no code change, but has a hardware limit and is a single point of failure. Horizontal = more machines behind a load balancer: scales much further and is fault tolerant, but the app must be stateless and data handling gets complex.

> [!question]- 🟢 Q3. Performance vs scalability?
> Performance problem: slow even for one user. Scalability problem: fast for one user but slow under heavy load. First is fixed with better code/queries, second with better architecture.

> [!question]- 🟢 Q4. What is a stateless server and why does it matter?
> A server that keeps no user data in its own memory between requests. Sessions live in Redis or a JWT, files in S3. Then any server can handle any request, so we can add or remove servers freely.

> [!question]- 🟡 Q5. What are sticky sessions and why avoid them?
> The load balancer always sends a user to the same server. It's a quick fix for stateful apps but causes uneven load, and users lose their session if that server dies. Better to make servers stateless.

> [!question]- 🟡 Q6. Scalability vs elasticity?
> Scalability = the system *can* grow by adding resources. Elasticity = it adds and removes resources *automatically* based on load (auto-scaling), which also saves cost when traffic drops.

> [!question]- 🟡 Q7. How do you scale a database?
> In order: optimize queries and indexes → add caching → read replicas for reads → bigger machine → partition by feature → shard by key for writes and huge data. Sharding last because it's the most complex.

> [!question]- 🟡 Q8. What's the downside of read replicas?
> Replication lag — a user writes to the primary, reads from a replica, and doesn't see their own change yet. Fix: read-your-own-writes from the primary for a short time, or for critical reads.

> [!question]- 🟡 Q9. What is sharding and its problems?
> Splitting rows across multiple databases by a key (e.g. user_id). Problems: cross-shard joins and transactions, rebalancing when adding shards, and hot shards when one key gets too much traffic. Choosing a good shard key is the key decision.

> [!question]- 🟡 Q10. How do message queues help scalability?
> They move slow work (emails, video processing) out of the request path so APIs respond fast, and they absorb spikes — workers process the backlog at their own speed, and you scale workers independently.

> [!question]- 🟡 Q11. What is the scale cube?
> Three ways to scale: X-axis = clone the same app behind a load balancer; Y-axis = split by function into services; Z-axis = split by data (sharding).

> [!question]- 🔴 Q12. You added more app servers but the site is still slow. Why?
> The bottleneck is probably elsewhere — usually the database (slow queries, connection limits, locks), or a shared dependency like one Redis or a slow third-party API. More app servers can even make it worse by sending more load to the DB. Measure first: DB metrics, p99 latency per dependency.

> [!question]- 🔴 Q13. How would you scale an app from 1 user to 10 million?
> One server → separate DB → load balancer + stateless app servers → cache + CDN → read replicas → queues + auto-scaling → sharding, services, multi-region. Each step fixes the current bottleneck; don't jump to the end.

> [!question]- 🔴 Q14. A flash sale is at 12:00 and traffic will be 20x. What do you do?
> Pre-scale servers before 12:00 (auto-scaling is too slow for a sudden spike), warm the cache with sale products, serve static pages via CDN, put orders on a queue, use rate limiting and a waiting room if needed, protect inventory with atomic DB updates or Redis counters, and load-test beforehand.

> [!question]- 🔴 Q15. Why use p99 latency instead of average?
> Averages hide the slow requests. p99 = 99% of requests are faster than this. Those slowest 1% are often your heaviest users (big carts, many followers), and at scale 1% is a lot of people.
