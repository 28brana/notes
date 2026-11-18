# 09 · Window Functions

> [!info] Topic info
> **Subject:** DBMS · **Part:** SQL & Data Modeling · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to DBMS](README.md) · [Interview questions →](questions/sql-and-modeling-qa.md)

> [!abstract] In one line
> A **window function** computes a value **across a set of related rows** (the "window") **without collapsing them** — so each row keeps its detail **and** gets a rank, running total, or previous/next value, defined by `OVER (PARTITION BY … ORDER BY …)`.

---

## 1. The problem

"Show each user's **top 3 most-liked posts**." GROUP BY gives one row per user — we lose the posts. We need to rank posts **within** each user while keeping every row. That's a window function.

---

## 2. How it works

```sql
function() OVER (
  PARTITION BY user_id      -- split rows into groups (like GROUP BY, but no collapse)
  ORDER BY likes DESC       -- order inside each group
  ROWS BETWEEN ...          -- optional frame: which rows count
)
```

```mermaid
flowchart LR
    R[📄 All posts] -- PARTITION BY user_id --> P[🗂️ One window per user]
    P -- ORDER BY likes DESC --> O[↕️ Sorted inside window]
    O -- function per row --> X[🏷️ Every row + its rank]
```

Window functions run **after** WHERE / GROUP BY / HAVING, so you can't filter on them directly — wrap in a CTE.

```sql
WITH ranked AS (
  SELECT user_id, post_id, likes,
         ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY likes DESC) AS rn
  FROM post_stats
)
SELECT * FROM ranked WHERE rn <= 3;    -- top 3 per user
```

---

## 3. ROW_NUMBER vs RANK vs DENSE_RANK

Posts with likes 90, 80, 80, 70:

| likes | **ROW_NUMBER** | **RANK** | **DENSE_RANK** |
|---|---|---|---|
| 90 | 1 | 1 | 1 |
| 80 | 2 | 2 | 2 |
| 80 | 3 | 2 | 2 |
| 70 | 4 | **4** (gap) | **3** (no gap) |

- **ROW_NUMBER** — always unique; ties broken arbitrarily. Use for "exactly N per group" or dedup.
- **RANK** — ties share a rank, then **skips** numbers (Olympic style).
- **DENSE_RANK** — ties share, **no gaps**. Use for "Nth highest value".

> [!tip] 🎤 Say it like this
> "Window functions are like GROUP BY that doesn't collapse rows — each row keeps its data and also gets a computed value over its partition. For ranking, ROW_NUMBER is always unique, RANK leaves gaps after ties, and DENSE_RANK doesn't. For top-N per group I use ROW_NUMBER in a CTE and filter `rn <= N`."

---

## 4. LAG / LEAD and running totals

```sql
SELECT day, signups,
  LAG(signups)  OVER (ORDER BY day) AS prev_day,          -- value from previous row
  LEAD(signups) OVER (ORDER BY day) AS next_day,          -- value from next row
  SUM(signups)  OVER (ORDER BY day) AS running_total,     -- cumulative
  AVG(signups)  OVER (ORDER BY day
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS avg_7d  -- moving average
FROM daily_signups;
```

- **LAG / LEAD** — compare with previous/next row: day-over-day growth, gaps between logins.
- **Running total** — `SUM() OVER (ORDER BY …)`.
- Others: `NTILE(4)` (quartiles), `FIRST_VALUE`, `LAST_VALUE`.

> [!warning] The default frame trap
> With `ORDER BY` and no frame, the default is `RANGE … CURRENT ROW` — **ties on the same day are added together**, and `LAST_VALUE` returns the current row, not the last. Write `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` when you mean the whole partition.

> [!failure] Interview traps
> - `WHERE ROW_NUMBER() OVER (...) <= 3` — not allowed; filter in an outer query/CTE.
> - Using RANK for "2nd highest salary" when duplicates exist — DENSE_RANK is safer.
> - Thinking window functions reduce row count — they never do.

---

## Summary

- `OVER (PARTITION BY … ORDER BY … frame)` — compute per row without collapsing.
- ROW_NUMBER (unique) · RANK (gaps) · DENSE_RANK (no gaps).
- LAG/LEAD for neighbors, `SUM() OVER` for running totals, frames for moving averages.

> [!success] Key takeaway
> "Top N per group", "Nth highest", "running total", "compare with previous row" — all four are window function questions.
