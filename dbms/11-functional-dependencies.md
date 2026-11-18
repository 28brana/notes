# 11 · Functional Dependencies

> [!info] Topic info
> **Subject:** DBMS · **Part:** SQL & Data Modeling · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to DBMS](README.md) · [Interview questions →](questions/sql-and-modeling-qa.md)

> [!abstract] In one line
> A **functional dependency (FD)** `A → B` means **knowing A fixes B** — if two rows have the same A, they must have the same B; FDs are the tool we use to **find keys** and decide **how to normalize** a table.

---

## 1. What it is

In `users(user_id, email, city)`: `user_id → email`. Give me a user ID and there's exactly one email.

But `city → user_id` is **not** an FD — many users live in Pune.

- **A** = determinant (left side), **B** = dependent (right side).
- **Trivial FD** — B is part of A: `{user_id, email} → email`. Always true, tells us nothing.

---

## 2. Types that matter for normalization

Take `post_likes(user_id, post_id, user_name, post_title, liked_at)` with key `{user_id, post_id}`:

| Type | Example | Problem it causes |
|---|---|---|
| **Full** | `{user_id, post_id} → liked_at` | ✅ None — needs the whole key |
| **Partial** | `user_id → user_name` | ❌ Depends on **part** of the key → breaks **2NF** |
| **Transitive** | In `posts(post_id, author_id, author_city)`: `post_id → author_id → author_city` | ❌ Non-key depends on non-key → breaks **3NF** |

---

## 3. Armstrong's axioms and closure

Three rules to derive new FDs:
- **Reflexivity** — if B ⊆ A, then A → B.
- **Augmentation** — if A → B, then AC → BC.
- **Transitivity** — if A → B and B → C, then A → C.

**Attribute closure (A⁺)** = everything A can determine. If A⁺ covers **all** columns, A is a **super key**; if it's also minimal, it's a **candidate key**.

> [!example] Worked example — find the key
> R(A, B, C, D), FDs: A → B, B → C, C → D.
> A⁺ = {A} → add B → add C → add D = **{A, B, C, D}**. All columns → **A is a candidate key**.
> B⁺ = {B, C, D} — missing A → not a key.
> Tip: an attribute that never appears on the **right side** of any FD must be in every key.

> [!tip] 🎤 Say it like this
> "A functional dependency A → B means A uniquely decides B. I compute attribute closures to find candidate keys, then look for partial dependencies, which break 2NF, and transitive ones, which break 3NF. That tells me exactly how to split the table."

> [!failure] Interview traps
> - Reading FDs from sample data — FDs come from **business rules**, not from what today's 10 rows happen to show.
> - Confusing partial (part of a composite key) with transitive (via a non-key column).

---

## Summary

- `A → B`: same A ⇒ same B.
- Full / partial / transitive — partial breaks 2NF, transitive breaks 3NF.
- Closure finds keys; Armstrong's axioms derive FDs.

> [!success] Key takeaway
> FDs are the "why" behind [normalization](12-normalization.md) — spot partial and transitive ones and you know what to split.
