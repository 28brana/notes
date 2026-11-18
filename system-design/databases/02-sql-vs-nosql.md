# 02 · SQL vs NoSQL

> [!info] Topic info
> **Subject:** System Design · **Chapter:** Databases · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Databases](README.md) · [Interview questions →](questions/databases-qa.md)

> [!abstract] In one line
> **SQL** databases give you a **fixed schema, joins, and ACID transactions**; **NoSQL** databases trade some of that for a **flexible schema** and **easy horizontal scaling** around a known access pattern.

---

## 1. The problem

Every design interview hits this question: "Which database would you use?"

Saying "MongoDB because it's fast" is a red flag. The interviewer wants to hear **trade-offs** tied to **your data and your queries**.

---

## 2. What the words mean

- **SQL (relational)** — data in tables with rows and columns. You query with SQL. Tables link via keys and **joins**. Examples: Postgres, MySQL.
- **NoSQL** — "not only SQL". A family of non-table models: [document](07-document-databases.md), [key-value](08-key-value-stores.md), [wide-column](10-wide-column-databases.md), [graph](12-graph-databases.md).

> [!note] NoSQL is not one thing
> Redis, MongoDB, and Cassandra are all "NoSQL" but behave very differently. Always name the **specific type**.

---

## 3. The real differences

| | SQL | NoSQL |
|---|---|---|
| **Schema** | Fixed, enforced by DB (schema-on-write) | Flexible, enforced by app (schema-on-read) |
| **Relations** | Joins, foreign keys | Usually no joins — denormalize or embed |
| **Transactions** | Full [ACID](03-acid-transactions.md), multi-row, multi-table | Often single-record only; some support more |
| **Scaling** | Vertical first; sharding is manual and hard | Built to shard across many nodes |
| **Consistency** | Strong by default | Often eventual or tunable (see [CAP](../core-concepts/07-cap-theorem.md)) |
| **Query flexibility** | Any ad-hoc query | Fast for designed queries, poor for others |
| **Design approach** | Model the data, then write queries | Model the **queries**, then shape the data |

The last row is the big one. In SQL you normalize and query any way later. In Cassandra or DynamoDB you must **know your queries first**.

---

## 4. When to pick which

**Pick SQL when:**
- Data is relational — users, orders, payments, follows.
- You need **transactions** (money, inventory, bookings).
- Queries will change and you need ad-hoc reporting.
- Scale fits one primary + read replicas (that's a lot — millions of users).

**Pick NoSQL when:**
- **Huge write volume** — 100K+ writes/sec of logs, events, messages.
- Access pattern is simple and known — "get by key", "get last 50 messages for chat X".
- Data shape varies a lot per record.
- You need multi-region, always-writable storage.

> [!example] Social app split
> 👤 Users, follows, payments → **Postgres**. 💬 Chat messages (billions, append-only, read by chat ID) → **Cassandra**. 🔑 Sessions → **Redis**.

> [!tip] 🎤 Say it like this
> "I'd default to a relational database because I get transactions, joins, and flexible queries for free. I'd move a specific piece to NoSQL only when its access pattern is simple and the write volume or scale outgrows a single primary — for example chat messages in Cassandra keyed by conversation ID."

> [!warning] The lines are blurring
> Postgres has JSONB (document-style), and can scale with Citus. MongoDB has multi-document transactions. **NewSQL** databases (CockroachDB, Spanner) give SQL + ACID + horizontal scale — at the cost of higher write latency.

> [!failure] Interview traps
> - "NoSQL is faster" — not in general. It's fast for the queries it was **designed** for.
> - "SQL can't scale" — read replicas, partitioning, and sharding take SQL very far (see [Sharding](../database-scaling/04-sharding.md)).
> - Forgetting that "schemaless" just moves the schema into your **app code**.

---

## Summary

| Need | Choose |
|---|---|
| **Transactions, joins, ad-hoc queries** | SQL |
| **Massive writes, simple known queries** | NoSQL (wide-column / key-value) |
| **Flexible nested records** | Document DB (or Postgres JSONB) |
| **SQL + global scale** | NewSQL (Spanner, CockroachDB) |

> [!success] Key takeaway
> SQL = flexible queries + strong guarantees. NoSQL = scale + speed for **known** access patterns. Default to SQL, justify any move away.
