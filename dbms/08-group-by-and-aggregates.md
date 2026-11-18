# 08 · GROUP BY & Aggregates

> [!info] Topic info
> **Subject:** DBMS · **Part:** SQL & Data Modeling · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to DBMS](README.md) · [Interview questions →](questions/sql-and-modeling-qa.md)

> [!abstract] In one line
> **GROUP BY** collapses rows into **one row per group**, **aggregate functions** (`COUNT, SUM, AVG, MIN, MAX`) summarize each group, and **HAVING** filters groups **after** aggregation — while **WHERE** filters rows **before**.

---

## 1. The problem

The dashboard needs "number of posts per user" and "users with more than 50 posts". Listing raw posts gives 2 million rows — we need one summarized row per user.

---

## 2. How it works

```mermaid
flowchart LR
    R[📄 All rows] -- WHERE filters rows --> F[🔍 Kept rows]
    F -- GROUP BY user_id --> G[🗂️ Buckets per user]
    G -- COUNT per bucket --> A[🔢 One row per user]
    A -- HAVING filters groups --> O[✅ Final result]
```

```sql
SELECT user_id, COUNT(*) AS post_count
FROM posts
WHERE created_at >= '2026-01-01'    -- rows: only this year's posts
GROUP BY user_id
HAVING COUNT(*) > 50                -- groups: only heavy posters
ORDER BY post_count DESC;
```

**Rule:** every column in `SELECT` must be either in `GROUP BY` or inside an aggregate. `SELECT user_id, content, COUNT(*) … GROUP BY user_id` fails — which `content` should it show?

---

## 3. WHERE vs HAVING

| | **WHERE** | **HAVING** |
|---|---|---|
| **Filters** | Individual rows | Groups |
| **Runs** | Before GROUP BY | After GROUP BY |
| **Aggregates allowed?** | ❌ No | ✅ Yes |
| **Performance** | Cheaper — fewer rows to group | Use only for aggregate conditions |

> [!tip] 🎤 Say it like this
> "WHERE filters rows before grouping, so it can't see aggregates. HAVING filters the groups after aggregation, so it can use COUNT or SUM. If a condition doesn't need an aggregate, I put it in WHERE — it shrinks the data before the expensive grouping step."

---

## 4. Aggregate details that get asked

- `COUNT(*)` counts rows; `COUNT(col)` skips NULLs; `COUNT(DISTINCT col)` counts unique values.
- `SUM`, `AVG`, `MIN`, `MAX` **ignore NULLs**. `AVG` of (10, NULL, 20) = **15**, not 10.
- Aggregates over zero rows: `COUNT` → 0, `SUM`/`AVG` → **NULL**. Wrap with `COALESCE(SUM(x), 0)`.
- **Conditional aggregation** — many counts in one pass:

```sql
SELECT user_id,
  COUNT(*) FILTER (WHERE created_at >= now() - interval '7 days') AS last_week,
  SUM(CASE WHEN content LIKE '%#sql%' THEN 1 ELSE 0 END)           AS sql_posts
FROM posts GROUP BY user_id;
```

> [!note] GROUP BY loses rows
> GROUP BY gives one row per group. If you need the summary **and** the detail rows (e.g. each post with its user's total), use [window functions](09-window-functions.md) instead.

> [!failure] Interview traps
> - Putting `COUNT(*) > 50` in WHERE.
> - Counting likes after joining posts + comments → inflated numbers; use `COUNT(DISTINCT)` or pre-aggregate.
> - Forgetting users with zero posts — need `LEFT JOIN` from `users` and `COUNT(p.post_id)`, not `COUNT(*)`.

---

## Summary

- GROUP BY → one row per group; aggregates summarize it.
- WHERE = rows, before · HAVING = groups, after.
- Aggregates ignore NULLs (except `COUNT(*)`).

> [!success] Key takeaway
> "Filter rows with WHERE, filter groups with HAVING" — plus the NULL rules — covers most aggregation questions.
