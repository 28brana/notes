# 03 · ACID Transactions

> [!info] Topic info
> **Subject:** System Design · **Chapter:** Databases · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Databases](README.md) · [Interview questions →](questions/databases-qa.md)

> [!abstract] In one line
> A **transaction** groups several reads and writes into **one unit** that either fully happens or doesn't; **ACID** is the set of guarantees it gives, and **isolation levels** decide how much concurrent transactions can see of each other.

---

## 1. The problem

A user buys Premium on our social app. We must (1) charge their wallet and (2) mark them Premium.

If the server crashes between step 1 and 2, they paid and got nothing. If two requests run at once, the wallet might be charged once but read twice. We need both steps to act as **one**.

---

## 2. ACID in one minute

| Letter | Meaning | Plain English |
|---|---|---|
| **A — Atomicity** | All or nothing | Crash halfway? Everything rolls back. |
| **C — Consistency** | Rules stay true | Constraints (balance ≥ 0, foreign keys) hold before and after. |
| **I — Isolation** | Concurrent txns don't step on each other | Each one acts *as if* it ran alone (to a degree — see below). |
| **D — Durability** | Committed = saved | Once you get "OK", it survives a crash. |

```sql
BEGIN;
UPDATE wallets SET balance = balance - 10 WHERE user_id = 42;
UPDATE users   SET plan = 'premium'       WHERE id = 42;
COMMIT;
```

**How it's built (one line each):**
- **Atomicity + Durability** → a **write-ahead log (WAL)**. Changes go to an append-only log on disk first. After a crash, the DB replays or undoes from the log.
- **Isolation** → **locks** or **MVCC** (multi-version concurrency control: readers see an older snapshot while writers create new versions, so reads don't block writes).

> [!note] Already covered
> The basics of ACID and durability are in [Reliability](../core-concepts/03-reliability.md). This note focuses on **isolation** — the part interviewers dig into.

---

## 3. The anomalies (what can go wrong)

When transactions run at the same time, these bugs can appear:

| Anomaly | What happens | Social app example |
|---|---|---|
| **Dirty read** | You read data another txn hasn't committed yet — and it rolls back | You see a post that then vanishes because its insert failed |
| **Non-repeatable read** | Read the same row twice in one txn, get different values | Like count reads 100, then 103 in the same report |
| **Phantom read** | Re-run the same *range* query, new rows appear | "Count posts today" returns 50, then 52 in the same txn |
| **Lost update** | Two txns read-modify-write the same value; one overwrites the other | Two likes read 100, both write 101 — one like is lost |
| **Write skew** | Two txns read the same data, each writes a different row, together breaking a rule | Two admins both see "2 admins left", both step down → 0 admins |

---

## 4. Isolation levels

Stronger isolation = fewer anomalies but **more locking/aborts and less throughput**.

| Level | Dirty read | Non-repeatable | Phantom | Lost update | Used by default in |
|---|---|---|---|---|---|
| **Read Uncommitted** | ❌ possible | ❌ | ❌ | ❌ | Almost nobody |
| **Read Committed** | ✅ prevented | ❌ | ❌ | ❌ | **Postgres**, Oracle, SQL Server |
| **Repeatable Read / Snapshot** | ✅ | ✅ | ⚠️ depends | ⚠️ depends | **MySQL InnoDB** |
| **Serializable** | ✅ | ✅ | ✅ | ✅ | Opt-in (CockroachDB default) |

> [!warning] Names lie
> Behaviour varies by database. Postgres Repeatable Read blocks phantoms and detects lost updates; MySQL's Repeatable Read does not always. Always check the DB docs.

### Fixing lost updates without Serializable

- **Atomic update:** `UPDATE posts SET likes = likes + 1 WHERE id = 7` — no read-modify-write in app code.
- **Pessimistic lock:** `SELECT ... FOR UPDATE` — lock the row until commit.
- **Optimistic lock:** add a `version` column; `UPDATE ... WHERE id=7 AND version=5`. If 0 rows changed, someone beat you — retry.

> [!tip] 🎤 Say it like this
> "ACID means a transaction is all-or-nothing, keeps constraints true, is isolated from others, and is durable once committed. Most databases default to Read Committed, which stops dirty reads but still allows lost updates. For counters I'd use an atomic increment; for read-then-write logic like booking a seat I'd use SELECT FOR UPDATE or optimistic locking with a version column."

> [!failure] Interview traps
> - Thinking "I use a transaction" means you're safe from lost updates — not at Read Committed.
> - Mixing up ACID **C** (constraints hold) with CAP **C** (all replicas agree). Different things!
> - Assuming ACID works across services — it doesn't. For that see [Distributed Transactions](../distributed-transactions/01-distributed-transactions-problem.md) and [Saga](../distributed-transactions/04-saga-pattern.md).

---

## Summary

- **A**tomic, **C**onsistent, **I**solated, **D**urable — built on WAL + locks/MVCC.
- Anomalies: dirty read, non-repeatable read, phantom, lost update, write skew.
- Default is usually **Read Committed**; go stronger only where correctness needs it.
- Fix lost updates with atomic updates, `FOR UPDATE`, or version checks.

> [!success] Key takeaway
> Know your isolation level. Most "random" data bugs under load are **lost updates** at Read Committed.
