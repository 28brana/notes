# 05 · Query Optimization

> [!info] Topic info
> **Subject:** System Design · **Chapter:** Database Scaling · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Database Scaling](README.md) · [Interview questions →](questions/database-scaling-qa.md)

> [!abstract] In one line
> **Query optimization** means **finding slow queries**, reading their **execution plan** with `EXPLAIN`, and fixing them — with indexes, **fewer round trips** (no N+1), **fetching less data**, and **keyset pagination**.

---

## 1. The problem

Our feed API went from 80 ms to **2 seconds** p99. The database CPU is high. Before buying bigger machines or sharding, check the **queries** — one bad query can burn more resources than 1,000 good ones.

---

## 2. Find, explain, fix

```mermaid
flowchart LR
    A[📊 Slow query log] --> B[🔍 EXPLAIN ANALYZE]
    B --> C[⚙️ Fix: index, rewrite, cache]
    C --> D[📊 Measure again]
```

1. **Find** — turn on the slow query log (e.g. log anything over 100 ms), or use `pg_stat_statements` to see which queries use the most **total** time.
2. **Explain** — `EXPLAIN ANALYZE` shows the plan the database picked and how long each step took.
3. **Fix** and **measure** again.

### What to look for in `EXPLAIN`

| You see | Meaning |
|---|---|
| **Seq Scan / full scan** on a big table | Missing or unused index |
| **Index Scan / Index Only Scan** | Good |
| **Rows estimated vs actual** far apart | Stale statistics — run `ANALYZE` |
| **Sort** on many rows | Index could provide order instead |
| **Nested loop** over large row counts | Join missing an index |

> [!tip] 🎤 Say it like this
> "First I find the slowest queries from the slow query log, then run EXPLAIN ANALYZE to see the plan. Usually it's a full table scan from a missing index, an N+1 pattern from the ORM, or deep offset pagination. I fix the biggest offender and measure again."

---

## 3. The N+1 problem

The classic ORM bug. Loading a feed of 20 posts with their authors:

```text
1 query:  SELECT * FROM posts LIMIT 20
20 queries: SELECT * FROM users WHERE id = ?   (one per post)
```

That's **21 round trips**. At 2 ms each = 42 ms, and it grows with page size.

Fix: fetch in **one or two** queries — a `JOIN`, or `WHERE id IN (…)` (ORMs call this eager loading, e.g. `include` / `select_related`). In GraphQL, use a **DataLoader** to batch.

---

## 4. Pagination: offset vs keyset

**Offset:** `LIMIT 20 OFFSET 100000` — the database still reads and throws away 100,000 rows. Page 5,000 is very slow. Rows also shift when new posts arrive, so users see duplicates.

**Keyset (cursor):** remember the last item seen and continue from there:

```sql
SELECT * FROM posts
WHERE created_at < :last_seen_time
ORDER BY created_at DESC LIMIT 20;
```

With an index on `created_at`, every page is equally fast.

| | Offset | Keyset / cursor |
|---|---|---|
| **Speed on deep pages** | Gets slower | Constant |
| **New rows inserted** | Duplicates/skips | Stable |
| **Jump to page 50** | ✅ Easy | ❌ Not possible |
| **Best for** | Admin tables, small data | Feeds, infinite scroll, APIs |

> [!note] Ties
> If two posts share the same `created_at`, add a tie-breaker: `ORDER BY created_at, id` and compare both.

---

## 5. Other quick wins

- **`SELECT` only needed columns** — not `SELECT *`; enables covering indexes.
- **Batch writes** — one insert of 500 rows beats 500 inserts.
- **Avoid functions on indexed columns** in `WHERE` (see [Indexing](01-indexing.md)).
- **Cache** hot, rarely changing results — see [Caching](../caching/01-what-is-caching.md).
- **Move heavy reports** to a replica or a warehouse, not the main DB.

> [!failure] Interview traps
> - Jumping to sharding before looking at slow queries and indexes.
> - Guessing instead of reading the `EXPLAIN` plan.
> - Using `OFFSET` for infinite scroll on a huge table.
> - Not noticing N+1 because each single query looks fast.

---

## Summary

- Find slow queries → `EXPLAIN ANALYZE` → fix → measure.
- Kill **N+1** with joins, `IN (…)`, or batching.
- Use **keyset pagination** for feeds and APIs.
- Fetch less, batch writes, push heavy reports elsewhere.

> [!success] Key takeaway
> The cheapest scaling is a better query. Always check the plan before adding hardware.
