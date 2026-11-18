# 10 · Views, Stored Procedures & Triggers

> [!info] Topic info
> **Subject:** DBMS · **Part:** SQL & Data Modeling · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to DBMS](README.md) · [Interview questions →](questions/sql-and-modeling-qa.md)

> [!abstract] In one line
> A **view** is a **saved query** that looks like a table, a **stored procedure** is **code stored and run inside the database**, and a **trigger** is code that runs **automatically** when rows are inserted, updated or deleted.

---

## 1. Views

Our analytics team keeps writing the same 3-table join for "post with author and like count". Save it once:

```sql
CREATE VIEW post_summary AS
SELECT p.post_id, u.name AS author, COUNT(l.user_id) AS likes
FROM posts p
JOIN users u  ON u.user_id = p.user_id
LEFT JOIN likes l ON l.post_id = p.post_id
GROUP BY p.post_id, u.name;
```

Now `SELECT * FROM post_summary WHERE likes > 100;` just works.

- **Why:** simplify queries, **hide columns** (give interns a view without `email`), keep a stable interface while tables change.
- A normal view stores **no data** — the query runs every time.
- **Updatable** only if simple (one table, no GROUP BY / DISTINCT / aggregates).

| | **View** | **Materialized view** |
|---|---|---|
| **Stores data?** | No, re-runs query | Yes, saved result |
| **Read speed** | Same as the query | Fast |
| **Freshness** | Always current | Stale until `REFRESH` |

More on the second one: [materialized views](../system-design/database-scaling/07-materialized-views.md).

---

## 2. Stored procedures and functions

```sql
CREATE PROCEDURE follow_user(p_from BIGINT, p_to BIGINT)
LANGUAGE plpgsql AS $$
BEGIN
  INSERT INTO follows VALUES (p_from, p_to) ON CONFLICT DO NOTHING;
  UPDATE users SET following_count = following_count + 1 WHERE user_id = p_from;
END $$;
-- CALL follow_user(1, 2);
```

- **Function** returns a value and can be used inside `SELECT`. **Procedure** is called with `CALL` and can manage transactions (in PostgreSQL 11+).

**✅ Pros** — one network round trip for many statements, logic close to data, can grant `EXECUTE` without table access.
**❌ Cons** — hard to version, test and debug; locks you to one database; business logic hidden from app developers.

---

## 3. Triggers

```sql
CREATE TRIGGER trg_like_count
AFTER INSERT ON likes
FOR EACH ROW EXECUTE FUNCTION bump_like_count();
```

- **BEFORE** triggers — validate or modify the row (e.g. set `updated_at`).
- **AFTER** triggers — side effects: audit logs, counters.
- **Row-level** (per row) vs **statement-level** (once per statement).

> [!warning] Triggers are invisible
> A developer runs one `INSERT` and 3 other tables change. A bulk insert of 1M likes fires the trigger 1M times. Keep triggers small — audit logs and `updated_at` are good uses; core business logic usually isn't.

> [!tip] 🎤 Say it like this
> "A view is a stored query — great for simplifying joins and hiding sensitive columns, but it doesn't store data unless it's materialized. Procedures run logic inside the DB to save round trips. Triggers fire automatically on insert, update or delete; I use them for audit logs or timestamps, not core business rules, because they're hidden and hard to debug."

> [!failure] Interview traps
> - Saying views make queries faster — a plain view doesn't; a materialized one does.
> - Putting heavy logic in triggers, then wondering why bulk imports are slow.

---

## Summary

- **View** = saved query, no data · **Materialized view** = saved result.
- **Procedure** = logic in the DB, `CALL`ed · **Function** = returns a value.
- **Trigger** = auto-run on DML; BEFORE / AFTER, row / statement.

> [!success] Key takeaway
> Views for simplicity and security, procedures for round trips, triggers sparingly — and always mention the maintainability cost.
