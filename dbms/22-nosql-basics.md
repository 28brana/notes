# 22 · NoSQL Basics

> [!info] Topic info
> **Subject:** DBMS · **Part:** Transactions & Internals · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to DBMS](README.md) · [Interview questions →](questions/transactions-and-internals-qa.md)

> [!abstract] In one line
> **NoSQL** databases drop fixed tables and joins for **flexible data models** (document, key-value, wide-column, graph) that **scale horizontally**, usually trading strict ACID for **BASE** and eventual consistency.

---

## 1. The problem

A chat app writes **200K messages/sec**. One Postgres primary can't take that, and the data is always read the same way: "last 50 messages of chat X".

We don't need joins here — we need **many machines** and a model shaped around one query.

---

## 2. The four types

| Type | Data looks like | Example | Use for |
|---|---|---|---|
| **Key-value** | `key → blob` | Redis, DynamoDB | Cache, sessions, counters |
| **Document** | JSON documents | MongoDB | Product catalog, user profiles |
| **Wide-column** | Row key → many columns, sorted | Cassandra, HBase | Huge write volume, time-series, chat |
| **Graph** | Nodes + edges | Neo4j | Friends-of-friends, recommendations |

Deeper per-type notes and the full comparison: [SQL vs NoSQL](../system-design/databases/02-sql-vs-nosql.md), [Database Types](../system-design/databases/01-database-types.md).

---

## 3. ACID vs BASE

**BASE** = **B**asically **A**vailable, **S**oft state, **E**ventual consistency.

| | **ACID** (SQL) | **BASE** (many NoSQL) |
|---|---|---|
| **Priority** | Correctness | Availability + scale |
| **Consistency** | Strong, immediate | Eventual — replicas catch up in ms–seconds |
| **Transactions** | Multi-row, multi-table | Often single-document/row only |

During a network split, a distributed DB must pick consistency or availability — see [CAP Theorem](../system-design/core-concepts/07-cap-theorem.md). Cassandra lets you tune it per query with **quorums**: if `R + W > N` (e.g. N=3, W=2, R=2), reads always see the latest write.

---

## 4. Modeling: queries first

- **SQL:** normalize, then write any query later.
- **NoSQL:** list your queries first, then **denormalize** data to answer each one in a single read.

```js
// MongoDB: embed what you read together
{ _id: "order_91", userId: "u42",
  items: [{ sku: "A1", qty: 2, price: 499 }],   // embedded, no join
  shipping: { city: "Pune", pin: "411001" } }
```

Embed when data is read together and bounded in size; **reference** (store an ID) when it's shared or grows without limit.

> [!tip] 🎤 Say it like this
> "NoSQL covers key-value, document, wide-column and graph databases. They give up joins and often multi-row transactions so they can shard easily across many nodes, and many are eventually consistent. I pick one when the access pattern is simple and known and the scale outgrows a single SQL primary — otherwise I default to Postgres."

> [!failure] Interview traps
> - "NoSQL is schemaless" — the schema just moves into your app code.
> - "NoSQL is faster" — only for the queries you designed it for; ad-hoc queries are painful.
> - Saying MongoDB has no transactions — it supports multi-document transactions now (with a cost).

---

## Summary

- 4 types: key-value, document, wide-column, graph.
- **BASE** vs **ACID**: availability and scale vs strict correctness.
- Model around **queries**; denormalize; embed vs reference.
- Tunable consistency with quorums: `R + W > N`.

> [!success] Key takeaway
> NoSQL isn't better or worse — it's a trade: you give up joins and flexibility to get horizontal scale for a known access pattern.
