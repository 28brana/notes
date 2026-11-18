# Q&A · Database Scaling

[← Back to Database Scaling](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 01 · Indexing

[Read the notes →](../01-indexing.md)

> [!question]- 🟢 Q1. What is a database index and what does it cost?
> An index is a sorted structure, usually a B-tree, over one or more columns with pointers back to the rows. It turns a full table scan into a log-time lookup. The cost is slower writes, because every insert, update, and delete must update each index, plus extra disk and memory.

> [!question]- 🟡 Q2. You have an index on (user_id, created_at). Which queries can use it?
> Queries filtering on `user_id`, or on `user_id` and `created_at`, can use it — and it also gives the `ORDER BY created_at` for free within a user. A query filtering only on `created_at` can't, because of the leftmost prefix rule: the index is sorted by `user_id` first. Put equality columns first and range or sort columns after.

> [!question]- 🟡 Q3. What is a covering index?
> It's an index that contains every column the query needs, so the database answers straight from the index without reading the table. That's called an index-only scan and saves a lot of random disk reads. You get it by adding the selected columns to the index.

> [!question]- 🔴 Q4. You added an index but the query is still slow and EXPLAIN shows a full scan. Why?
> Common reasons: the query wraps the column in a function like `LOWER(email)`, uses a leading wildcard `LIKE '%x'`, has a type mismatch, or skips the leftmost column of a composite index. Or the column has low selectivity, so the planner decides a scan is cheaper. It could also be stale statistics — I'd run `ANALYZE` and check the plan again.

---

## 02 · Partitioning

[Read the notes →](../02-partitioning.md)

> [!question]- 🟢 Q5. What's the difference between partitioning and sharding?
> Partitioning splits a table into pieces inside the same database server; sharding splits data across different servers. Partitioning speeds up queries and makes archiving easy but adds no capacity. Sharding adds capacity for writes and storage but makes joins, transactions, and routing harder.

> [!question]- 🟡 Q6. What is partition pruning and when does it fail?
> Pruning is when the database skips partitions that can't match the query, like only scanning November when you filter by a November date. It fails when the query doesn't filter on the partition key — then every partition gets scanned, which can be slower than one big table.

> [!question]- 🔴 Q7. Deleting old rows from a 2-billion-row logs table takes hours and locks things up. What do you do?
> Partition the table by time, say monthly. Then removing old data is just dropping old partitions, which is instant and doesn't touch live data. As a bonus, queries on recent data only scan recent partitions.

---

## 03 · Replication

[Read the notes →](../03-replication.md)

> [!question]- 🟢 Q8. Why replicate a database, and does it help with write scaling?
> Replication gives more read capacity, high availability through failover, and lower latency by placing copies near users. It does not help write scaling, because every replica still has to apply every write. For writes you need sharding.

> [!question]- 🟡 Q9. Compare leader-follower, multi-leader, and leaderless replication.
> Leader-follower sends all writes to one leader and reads to followers — simple, no conflicts, the default. Multi-leader lets several leaders, usually one per region, accept writes, which lowers write latency but creates write conflicts. Leaderless writes and reads from several nodes using quorums, so there's no failover, but consistency is weaker.

> [!question]- 🟡 Q10. What does W + R > N mean in quorum replication?
> With N copies, a write waits for W acknowledgements and a read asks R nodes. If W plus R is greater than N, the read set and write set always overlap, so at least one node in the read has the latest value. For example N = 3, W = 2, R = 2.

> [!question]- 🔴 Q11. A user updates their profile, refreshes, and sees the old data. What's happening and how do you fix it?
> The read went to a replica that hasn't caught up yet — replication lag breaking read-your-writes consistency. Fix it by reading the user's own data from the leader, or reading from the leader for a few seconds after a write, or only routing to replicas that have caught up to the user's last write position.

---

## 04 · Sharding

[Read the notes →](../04-sharding.md)

> [!question]- 🟢 Q12. What is sharding and when should you do it?
> Sharding splits data across several database servers by a shard key, so each handles part of the writes and storage. Do it only when one machine can't handle the write load or data size, after indexing, caching, vertical scaling, and replicas. It adds a lot of complexity.

> [!question]- 🟡 Q13. How do you choose a shard key?
> It should have high cardinality, spread both data and traffic evenly, and match the main access pattern so most queries hit one shard. For a social app, `user_id` works well since a user's data lives together. Bad choices are timestamps, which make the newest shard hot, and low-cardinality fields like country.

> [!question]- 🟡 Q14. Compare range, hash, and directory-based sharding.
> Range sharding is good for range queries but creates hot spots on new ranges. Hash sharding spreads load evenly but makes range queries hit every shard, and plain modulo moves most data when you add shards — consistent hashing fixes that. Directory-based uses a lookup table for full control, but the table is an extra hop and a possible single point of failure.

> [!question]- 🔴 Q15. One shard is at 100% CPU while the others are idle. What do you do?
> It's a hot shard, often a celebrity or viral key, or a bad shard key. Short term, cache the hot data and add replicas for that shard's reads. Longer term, split the hot key with a suffix, move hot users to a dedicated shard, or rethink the shard key. Starting with many logical shards makes moving load much easier.

---

## 05 · Query Optimization

[Read the notes →](../05-query-optimization.md)

> [!question]- 🟢 Q16. How do you find and fix a slow query?
> Find it with the slow query log or `pg_stat_statements`, sorted by total time. Run `EXPLAIN ANALYZE` to see the plan — look for full scans, big sorts, and bad row estimates. Fix with an index or a rewrite, then measure again.

> [!question]- 🟡 Q17. What is the N+1 query problem?
> It's when code runs one query to fetch a list, then one more query per item — like loading 20 posts and then 20 separate author queries. Each looks fast, but the round trips add up. Fix it with a join, a single `WHERE id IN (...)` query, eager loading in the ORM, or a DataLoader in GraphQL.

> [!question]- 🔴 Q18. Infinite scroll gets slower the further users scroll, and sometimes shows duplicates. Why?
> It's using offset pagination — the database reads and throws away all skipped rows, so deep pages are slow, and new posts shift rows, causing duplicates. Switch to keyset pagination: pass a cursor like the last `created_at` and `id`, and query `WHERE (created_at, id) < cursor`. With an index, every page is equally fast and stable.

---

## 06 · Denormalization

[Read the notes →](../06-denormalization.md)

> [!question]- 🟢 Q19. What is denormalization and why would you do it?
> It's storing duplicate or precomputed data on purpose, like the author's name or a like count on the post row. It avoids joins and expensive counts on hot read paths, and it's often necessary after sharding when joins aren't possible. The cost is more work and risk on writes.

> [!question]- 🟡 Q20. How do you keep denormalized copies consistent?
> If both are in one database, update them in the same transaction. Across services or shards, publish an event and let a worker update the copies — using the outbox pattern so events aren't lost. Add a periodic reconciliation job to fix any drift, and accept short staleness where it doesn't matter.

> [!question]- 🔴 Q21. A viral post's like counter is causing lock contention and slow writes. Fix it.
> Thousands of updates hit one row, so they queue up on its lock. Buffer likes in Redis or a queue and flush to the database in batches, or split the counter into several rows and sum them on read. Users don't need an exact like count in real time.

---

## 07 · Materialized Views

[Read the notes →](../07-materialized-views.md)

> [!question]- 🟢 Q22. What's the difference between a view and a materialized view?
> A view is just a saved query — it runs again every time you read it, so it's not faster. A materialized view stores the query's result like a table, so reads are fast. The trade-off is that the data is only as fresh as the last refresh.

> [!question]- 🟡 Q23. How do you keep a materialized view fresh?
> A full refresh on a schedule is simplest but costly on big data. Incremental refresh applies only the changes, and streaming updates the view as each change arrives, for near real-time. In PostgreSQL, use a concurrent refresh so readers aren't blocked.

> [!question]- 🔴 Q24. A dashboard query takes 10 seconds and runs on every page load. What do you do?
> First check if the query can be optimized with indexes. If it's an inherently heavy aggregate and slight staleness is okay, precompute it as a materialized view refreshed every few minutes, or cache the result. If it needs to be near real-time, maintain the aggregate incrementally from a change stream.

---

## 08 · Connection Pooling

[Read the notes →](../08-connection-pooling.md)

> [!question]- 🟢 Q25. Why use a connection pool?
> Opening a database connection costs a TCP and TLS handshake plus authentication, often tens of milliseconds, and each connection uses memory on the server. A pool keeps a few connections open and reuses them, so requests are faster and the database isn't overwhelmed by thousands of connections.

> [!question]- 🟡 Q26. How big should a connection pool be?
> Smaller than people expect — a database can only really run about as many queries at once as it has cores. A common starting point is cores times two plus disks, so around 20 for an 8-core DB, then load test. And remember the total is instances times pool size.

> [!question]- 🔴 Q27. After moving to serverless, the database starts rejecting connections during traffic spikes. Why, and how do you fix it?
> Each function instance opens its own connection, so a spike to a thousand instances blows past `max_connections` — a connection storm. Put a pooler like PgBouncer or RDS Proxy in front, reuse connections across invocations, and cap function concurrency. Raising `max_connections` alone just makes the database slower.
