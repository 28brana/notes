# System Design · Database Scaling

[← Back to System Design](../README.md) · [Interview Q&A →](questions/database-scaling-qa.md) · [Practice questions 📝](questions/practice-questions.md)

The database is usually the first thing to break as an app grows. This chapter covers the toolbox — from cheap fixes (indexes, better queries) to heavy ones (sharding) — and interviewers expect you to know **which to reach for, and in what order**.

| # | Topic | In one line | Priority |
|---|---|---|---|
| 01 | [Indexing](01-indexing.md) | Sorted B-tree lookups instead of full table scans; composite, covering, write cost | 🔥 High |
| 02 | [Partitioning](02-partitioning.md) | Split a big table into pieces on one server; partition pruning | ⚡ Medium |
| 03 | [Replication](03-replication.md) | Copies on many machines: leader-follower, multi-leader, leaderless; replication lag | 🔥 High |
| 04 | [Sharding](04-sharding.md) | Split data across servers by a shard key; hot shards, resharding, cross-shard queries | 🔥 High |
| 05 | [Query Optimization](05-query-optimization.md) | EXPLAIN, N+1, offset vs keyset pagination | ⚡ Medium |
| 06 | [Denormalization](06-denormalization.md) | Duplicate data on purpose to skip joins; keep copies in sync | ⚡ Medium |
| 07 | [Materialized Views](07-materialized-views.md) | Store an expensive query's result and refresh it | ⚡ Medium |
| 08 | [Connection Pooling](08-connection-pooling.md) | Reuse a few DB connections; PgBouncer, pool sizing, serverless storms | ⚡ Medium |

> [!important] The usual order of fixes
> Optimize queries + indexes → cache → vertical scale → read replicas → partition → denormalize / materialized views → **shard last**.
