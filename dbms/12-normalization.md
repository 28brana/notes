# 12 · Normalization

> [!info] Topic info
> **Subject:** DBMS · **Part:** SQL & Data Modeling · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to DBMS](README.md) · [Interview questions →](questions/sql-and-modeling-qa.md)

> [!abstract] In one line
> **Normalization** splits a table into smaller related tables so **each fact is stored once**, removing **insert, update and delete anomalies** — step by step through **1NF → 2NF → 3NF → BCNF** using [functional dependencies](11-functional-dependencies.md).

---

## 1. The problem

An intern stores everything in one sheet-like table:

| post_id | title | author_id | author_name | tags |
|---|---|---|---|---|
| 101 | Joins 101 | 1 | Asha | sql, dbms |
| 102 | Indexes | 1 | Asha | dbms |

Three **anomalies** show up:
- **Update** — Asha renames herself; we must change 500 rows. Miss one → two different names.
- **Insert** — a new user with no posts can't be stored (no `post_id`).
- **Delete** — delete Asha's only post and we lose that Asha exists.

---

## 2. Step by step

```mermaid
flowchart LR
    U[📄 Raw table] -- atomic values --> N1[1️⃣ 1NF]
    N1 -- remove partial deps --> N2[2️⃣ 2NF]
    N2 -- remove transitive deps --> N3[3️⃣ 3NF]
    N3 -- every determinant is a key --> B[⭐ BCNF]
```

### 1NF — atomic values, no repeating groups

`tags = "sql, dbms"` holds a list. Split to one value per cell:

`post_tags_raw(post_id, tag, title, author_id, author_name)` — key **{post_id, tag}**.

### 2NF — 1NF + no partial dependency

`title`, `author_id`, `author_name` depend only on `post_id` — **part** of the key. Split:
- `posts(post_id, title, author_id, author_name)`
- `post_tags(post_id, tag)`

(A table with a single-column key is automatically in 2NF.)

### 3NF — 2NF + no transitive dependency

In `posts`: `post_id → author_id → author_name`. A non-key column depends on another non-key column. Split:
- `posts(post_id, title, author_id)`
- `users(author_id, author_name)`

Now Asha's name lives in **one row**. All three anomalies are gone.

### BCNF — every determinant must be a super key

3NF still allows a gap. Example: `group_mods(user_id, group_id, moderator_id)`. Rules: each moderator handles **one** group; each user has **one** moderator per group.
- FDs: `{user_id, group_id} → moderator_id` and `moderator_id → group_id`.
- `moderator_id` is not a super key, but it determines `group_id` → **violates BCNF** (3NF tolerates it because `group_id` is part of a candidate key).
- Split: `moderators(moderator_id, group_id)` + `user_mods(user_id, moderator_id)`.

> [!warning] BCNF's cost
> The BCNF split can lose a dependency — "one moderator per user per group" can no longer be checked by a single table constraint. That's why many designs **stop at 3NF**, which always keeps dependencies.

> [!tip] 🎤 Say it like this
> "Normalization removes redundancy so each fact lives in one place. 1NF means atomic values, 2NF removes partial dependencies on a composite key, 3NF removes transitive dependencies between non-key columns, and BCNF says every determinant must be a super key. Most production schemas aim for 3NF and then denormalize specific hot paths on purpose."

---

## 3. Quick cheat sheet

| Form | Rule | Fixes |
|---|---|---|
| **1NF** | Atomic values, unique rows | Lists in cells |
| **2NF** | No non-key column depends on **part** of a key | Partial dependency |
| **3NF** | No non-key → non-key dependency | Transitive dependency |
| **BCNF** | Left side of every FD is a super key | Leftover 3NF anomalies |
| **4NF / 5NF** | No independent multi-valued / join dependencies | Rare — know the names |

---

## 4. When to denormalize

Normalized = clean writes, but reads need joins. At 20,000 feed reads/s, joining posts + users + likes each time is too slow.

So we **denormalize on purpose**: store `like_count` on `posts`, copy `author_name` into a feed table. Faster reads, harder writes, risk of stale copies. Full details: [Denormalization](../system-design/database-scaling/06-denormalization.md).

> [!success] In real life
> **Normalize first (3NF), measure, then denormalize only the hot read paths** — and keep the normalized tables as the source of truth.

> [!failure] Interview traps
> - Saying "higher normal form is always better" — more tables = more joins.
> - Confusing 2NF (partial, needs a composite key) with 3NF (transitive).
> - Calling denormalization "bad design" — it's a deliberate read-vs-write trade-off.

---

## Summary

| Step | Removes |
|---|---|
| **1NF** | Multi-valued cells |
| **2NF** | Partial dependencies |
| **3NF** | Transitive dependencies |
| **BCNF** | Non-key determinants |

> [!success] Key takeaway
> "The key, the whole key, and nothing but the key" — 1NF, 2NF, 3NF in one line. Normalize for correctness, denormalize for speed.
