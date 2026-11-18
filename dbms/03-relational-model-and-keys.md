# 03 · Relational Model & Keys

> [!info] Topic info
> **Subject:** DBMS · **Part:** SQL & Data Modeling · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to DBMS](README.md) · [Interview questions →](questions/sql-and-modeling-qa.md)

> [!abstract] In one line
> The **relational model** stores data as **tables (relations)** of **rows (tuples)** and **columns (attributes)**, and **keys** are the columns that **uniquely identify rows** and **link tables together**.

---

## 1. The vocabulary

| Formal term | Everyday term | Example |
|---|---|---|
| **Relation** | Table | `users` |
| **Tuple** | Row | `(42, 'asha', 'asha@x.com')` |
| **Attribute** | Column | `email` |
| **Domain** | Allowed values / type | `email` is VARCHAR(255) |
| **Degree** | Number of columns | 3 |
| **Cardinality** | Number of rows | 10 million |

In theory a relation is a **set**: no duplicate rows and no row order. SQL tables relax this — they allow duplicates unless you add a key.

---

## 2. The keys, using our `users` table

`users(user_id, email, phone, name)` — `user_id`, `email` and `phone` are each unique.

- **Super key** — *any* set of columns that uniquely identifies a row. `{user_id}`, `{user_id, name}`, `{email, phone}` … many of them.
- **Candidate key** — a **minimal** super key (remove any column and it stops being unique). Here: `{user_id}`, `{email}`, `{phone}`.
- **Primary key** — the one candidate key we **choose**. Unique + NOT NULL, one per table. We pick `user_id`.
- **Alternate key** — the candidate keys we didn't pick: `email`, `phone` → enforce with `UNIQUE`.
- **Composite key** — a key made of 2+ columns: `likes(user_id, post_id)`.
- **Foreign key** — a column pointing to another table's key: `posts.user_id → users.user_id`.
- **Surrogate key** — an artificial ID with no business meaning (auto-increment, UUID). **Natural key** — a real-world value (email, ISBN).

```mermaid
flowchart LR
    SK[🔑 Super keys - any unique set] --> CK[🎯 Candidate keys - minimal]
    CK --> PK[⭐ Primary key - chosen one]
    CK --> AK[🔁 Alternate keys - the rest]
```

> [!tip] 🎤 Say it like this
> "A super key is any column set that uniquely identifies a row. A candidate key is a minimal super key. We pick one candidate as the primary key, and the others become alternate keys with UNIQUE constraints. A foreign key links a row to a primary key in another table."

---

## 3. Integrity constraints

```sql
CREATE TABLE posts (
  post_id    BIGSERIAL PRIMARY KEY,                -- entity integrity
  user_id    BIGINT NOT NULL REFERENCES users(user_id)
               ON DELETE CASCADE,                  -- referential integrity
  content    TEXT NOT NULL CHECK (length(content) <= 5000),  -- domain
  created_at TIMESTAMP DEFAULT now()
);
```

- **Entity integrity** — PK can't be NULL.
- **Referential integrity** — FK must point to an existing row (or be NULL).
- **ON DELETE** options: `CASCADE` (delete posts too), `SET NULL`, `RESTRICT` (block the delete).

> [!note] Surrogate vs natural key
> Prefer a **surrogate** PK: emails change, and a 16-byte UUID or 8-byte BIGINT is smaller to index and join on than a long string. Keep the natural key unique with a `UNIQUE` constraint.

> [!failure] Interview traps
> - "Primary key and unique key are the same" — PK is NOT NULL and one per table; UNIQUE allows NULLs (usually) and many per table.
> - Calling `{user_id, name}` a candidate key — it's a super key, not minimal.
> - Thinking an FK must be unique — many posts share the same `user_id`.

---

## Summary

- Relation = table, tuple = row, attribute = column.
- **Super ⊇ Candidate ⊇ Primary**; leftover candidates = alternate keys.
- Composite = multi-column; surrogate = artificial ID; foreign = link.
- Constraints: entity, referential, domain (CHECK, NOT NULL).

> [!success] Key takeaway
> Keys are how a relational database guarantees "no duplicates" and "no broken links" — know the hierarchy super → candidate → primary cold.
