# 07 · Subqueries & CTEs

> [!info] Topic info
> **Subject:** DBMS · **Part:** SQL & Data Modeling · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to DBMS](README.md) · [Interview questions →](questions/sql-and-modeling-qa.md)

> [!abstract] In one line
> A **subquery** is a query inside another query; a **correlated** subquery re-runs **once per outer row**; a **CTE** (`WITH …`) gives a subquery a **name** so long queries read top-to-bottom — and a **recursive CTE** walks hierarchies like comment threads.

---

## 1. The problem

"Show posts that have more likes than the average post." That needs two steps: compute the average, then compare each post to it. Subqueries and CTEs let us write multi-step logic in one query.

---

## 2. Types of subqueries

| Type | Returns | Example use |
|---|---|---|
| **Scalar** | One value | `WHERE likes > (SELECT AVG(...) ...)` |
| **Row / list** | One column, many rows | `WHERE user_id IN (SELECT ...)` |
| **Table (derived table)** | A full table | `FROM (SELECT ...) AS t` |
| **Correlated** | Depends on the outer row | `WHERE EXISTS (SELECT 1 ... WHERE x = outer.x)` |

**Non-correlated** runs once. **Correlated** logically runs per outer row — 1M users = 1M executions, unless the optimizer rewrites it as a join.

```sql
-- Correlated: users who commented on their own post
SELECT u.name FROM users u
WHERE EXISTS (
  SELECT 1 FROM comments c JOIN posts p ON p.post_id = c.post_id
  WHERE c.user_id = u.user_id AND p.user_id = u.user_id
);
```

> [!warning] NOT IN with NULLs
> `WHERE user_id NOT IN (SELECT user_id FROM posts)` returns **zero rows** if any `posts.user_id` is NULL. Use `NOT EXISTS` — it's NULL-safe.

---

## 3. CTEs — named steps

```sql
WITH post_likes AS (
  SELECT post_id, COUNT(*) AS likes
  FROM likes GROUP BY post_id
)
SELECT post_id, likes
FROM post_likes
WHERE likes > (SELECT AVG(likes) FROM post_likes);
```

`post_likes` is defined once and used twice. Much easier to read than nesting.

> [!note] Is a CTE faster?
> Usually no — it's mainly for **readability**. Modern PostgreSQL (12+) inlines CTEs like subqueries. Older Postgres always materialized them, which could be slower or faster depending on the query.

---

## 4. Recursive CTE — trees and threads

Comments with replies (`comments.parent_id`) form a tree. Fetch a whole thread:

```sql
WITH RECURSIVE thread AS (
  SELECT comment_id, parent_id, body, 1 AS depth
  FROM comments WHERE comment_id = 500          -- anchor: root
  UNION ALL
  SELECT c.comment_id, c.parent_id, c.body, t.depth + 1
  FROM comments c JOIN thread t ON c.parent_id = t.comment_id  -- step
)
SELECT * FROM thread;
```

Anchor runs once, then the recursive part repeats until it adds no new rows. Also used for org charts, category trees, and "friends of friends".

> [!tip] 🎤 Say it like this
> "A subquery nests one query in another. A correlated one references the outer row, so it logically runs per row — I'd prefer EXISTS or a join there. CTEs are named subqueries that make multi-step queries readable, and recursive CTEs let me traverse trees like comment threads in one query."

> [!failure] Interview traps
> - Using `NOT IN` against a nullable column.
> - Claiming CTEs always improve performance.
> - Recursive CTE without a stop condition on cyclic data (e.g. follows) → infinite loop; add a depth limit.

---

## Summary

- Scalar, list, table, and correlated subqueries.
- Prefer `EXISTS` / `NOT EXISTS` for existence checks.
- CTE = readable named step; recursive CTE = hierarchies.

> [!success] Key takeaway
> Break hard SQL problems into CTE steps out loud — interviewers reward the clear thinking as much as the final query.
