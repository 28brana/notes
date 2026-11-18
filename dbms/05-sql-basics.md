# 05 · SQL Basics

> [!info] Topic info
> **Subject:** DBMS · **Part:** SQL & Data Modeling · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to DBMS](README.md) · [Interview questions →](questions/sql-and-modeling-qa.md)

> [!abstract] In one line
> **SQL** is the language for relational databases — you say **what** data you want, not **how** to get it — and a query runs in a **fixed logical order** (`FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`) that's different from how you write it.

---

## 1. Our running schema

Used in every SQL note in this subject:

```sql
users    (user_id PK, name, email UNIQUE, city, created_at)
posts    (post_id PK, user_id FK, content, created_at)
comments (comment_id PK, post_id FK, user_id FK, body, created_at)
likes    (user_id FK, post_id FK, created_at, PK(user_id, post_id))
follows  (follower_id FK, followee_id FK, PK(follower_id, followee_id))
```

---

## 2. Command groups

| Group | Commands | Purpose |
|---|---|---|
| **DDL** | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` | Structure |
| **DML** | `SELECT`, `INSERT`, `UPDATE`, `DELETE` | Data |
| **DCL** | `GRANT`, `REVOKE` | Permissions |
| **TCL** | `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT` | Transactions |

---

## 3. Order of execution

You **write** `SELECT … FROM … WHERE …`, but the database **runs** it like this:

```mermaid
flowchart LR
    F[📦 FROM and JOIN] --> W[🔍 WHERE] --> G[🗂️ GROUP BY] --> H[✅ HAVING] --> S[📋 SELECT] --> D[🧹 DISTINCT] --> O[↕️ ORDER BY] --> L[✂️ LIMIT]
```

This explains classic errors:
- You **can't** use a `SELECT` alias in `WHERE` — `WHERE` runs first.
- You **can** use it in `ORDER BY` — that runs after `SELECT`.
- You **can't** use `COUNT(*)` in `WHERE` — groups don't exist yet. Use `HAVING`.

```sql
SELECT city, COUNT(*) AS total
FROM users
WHERE created_at >= '2026-01-01'   -- filter rows first
GROUP BY city
HAVING COUNT(*) > 100              -- filter groups
ORDER BY total DESC                -- alias OK here
LIMIT 5;
```

> [!tip] 🎤 Say it like this
> "SQL is declarative — I describe the result, the optimizer picks the plan. Logically it runs FROM and joins first, then WHERE, GROUP BY, HAVING, then SELECT, ORDER BY and LIMIT. That's why I can't use a select alias or an aggregate in WHERE."

---

## 4. Must-know comparisons

| Pair | Difference |
|---|---|
| **DELETE vs TRUNCATE vs DROP** | `DELETE` removes chosen rows (row-by-row, fires triggers, `WHERE` allowed). `TRUNCATE` empties the whole table fast and resets identity. `DROP` removes the table itself |
| **UNION vs UNION ALL** | `UNION` removes duplicates (extra sort/hash). `UNION ALL` keeps all rows — faster |
| **CHAR vs VARCHAR** | `CHAR(10)` always stores 10, padded. `VARCHAR(10)` stores up to 10 |
| **IN vs EXISTS** | `IN` compares against a list of values. `EXISTS` just checks if any row matches and stops at the first one |

> [!warning] NULL is not a value
> `NULL = NULL` is **unknown**, not true. Use `IS NULL`. `COUNT(col)` skips NULLs; `COUNT(*)` doesn't. `NOT IN (subquery)` returns **nothing** if the subquery has a NULL — use `NOT EXISTS`.

> [!failure] Interview traps
> - Using an aggregate in `WHERE` instead of `HAVING`.
> - Saying `TRUNCATE` can never roll back — in PostgreSQL it can inside a transaction; in MySQL it can't.
> - `SELECT *` in production — fetches unused columns and breaks when schemas change.

---

## Summary

- DDL / DML / DCL / TCL.
- Execution order: **FROM → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT**.
- NULL logic is three-valued: true / false / unknown.

> [!success] Key takeaway
> If you remember the execution order, half of "why does this query error?" questions answer themselves.
