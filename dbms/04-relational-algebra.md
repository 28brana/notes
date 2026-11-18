# 04 · Relational Algebra

> [!info] Topic info
> **Subject:** DBMS · **Part:** SQL & Data Modeling · **Priority:** 💤 Low · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to DBMS](README.md) · [Interview questions →](questions/sql-and-modeling-qa.md)

> [!abstract] In one line
> **Relational algebra** is the **math behind SQL** — a small set of operators (**select, project, join, union, difference**…) that take tables in and give a table out; the database turns your SQL into these operators to plan and optimize it.

---

## 1. Why it matters

Mostly a college / GATE topic. In interviews it shows up as "what does SQL compile to?" — the query optimizer rewrites a tree of these operators to find a faster plan (see [query processing](21-query-processing.md)).

---

## 2. The operators mapped to SQL

| Operator | Symbol | What it does | SQL equivalent |
|---|---|---|---|
| **Selection** | σ | Filter rows | `WHERE city = 'Pune'` |
| **Projection** | π | Pick columns (removes duplicates) | `SELECT DISTINCT name` |
| **Rename** | ρ | Rename table/column | `AS` |
| **Union** | ∪ | Rows in A or B | `UNION` |
| **Difference** | − | Rows in A not in B | `EXCEPT` |
| **Intersection** | ∩ | Rows in both | `INTERSECT` |
| **Cartesian product** | × | Every row × every row | `CROSS JOIN` |
| **Join** | ⋈ | Product + condition | `JOIN … ON` |
| **Division** | ÷ | "For all" queries | `NOT EXISTS … NOT EXISTS` |

Union, difference and intersection need **union-compatible** tables: same number of columns with matching types.

Example — names of users in Pune:
`π name ( σ city='Pune' (users) )` → `SELECT DISTINCT name FROM users WHERE city = 'Pune';`

> [!example] Division in plain English
> "Users who liked **every** post by user 7." Division answers *for-all* questions. In SQL it's usually written as double `NOT EXISTS` or `GROUP BY … HAVING COUNT = total`.

> [!tip] 🎤 Say it like this
> "Relational algebra is the procedural math SQL is built on. Selection filters rows, projection picks columns, and join combines tables. The optimizer turns SQL into an algebra tree and reorders it — for example pushing selections down so it filters before joining."

> [!failure] Interview traps
> - Mixing up **selection (σ = rows)** and **projection (π = columns)** — SQL's `SELECT` keyword is actually projection.
> - Forgetting projection removes duplicates in algebra, but SQL `SELECT` doesn't unless you say `DISTINCT`.

---

## Summary

- σ filters rows, π picks columns, ⋈ joins, ∪ / − / ∩ are set operations.
- Every SQL query maps to an algebra tree the optimizer can rearrange.

> [!success] Key takeaway
> Know σ vs π and that "push selections down before joins" is the classic optimization.
