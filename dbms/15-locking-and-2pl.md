# 15 · Locking & Two-Phase Locking (2PL)

> [!info] Topic info
> **Subject:** DBMS · **Part:** Transactions & Internals · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to DBMS](README.md) · [Interview questions →](questions/transactions-and-internals-qa.md)

> [!abstract] In one line
> Databases use **shared (S)** locks for reads and **exclusive (X)** locks for writes; **two-phase locking** — acquire all, then release — guarantees **serializable** schedules, and **strict 2PL** also prevents cascading aborts.

---

## 1. The problem

Two users book the last seat on a flight at the same time. Both read `seats_left = 1`, both write `0`, both get a ticket.

We need a way to make the second transaction **wait** until the first one is done with that row.

---

## 2. Shared vs exclusive locks

| | **S held by other** | **X held by other** |
|---|---|---|
| **Request S** (read) | ✅ granted — many readers OK | ❌ wait |
| **Request X** (write) | ❌ wait | ❌ wait |

- **Shared (S):** "I'm reading — others may read too, nobody writes."
- **Exclusive (X):** "I'm writing — nobody else reads or writes."
- **Lock upgrade:** S → X when a txn reads then decides to write (a classic deadlock source — see [Deadlocks](17-deadlocks.md)).

**Granularity:** lock a row, a page, or the whole table. Row locks = more concurrency but more lock overhead. **Intention locks** (IS, IX) on the table say "I hold row locks inside" so a table-level lock request can be checked quickly.

---

## 3. Two-phase locking (2PL)

Just taking locks isn't enough — if you unlock too early, another txn can sneak in between. 2PL adds one rule: **once you release any lock, you can't take a new one.**

```mermaid
flowchart LR
    A[🔒 Growing phase: take locks only] -- lock point --> B[🔓 Shrinking phase: release only]
```

- **Basic 2PL** → guarantees **conflict serializability**.
- But it allows **cascading rollbacks**: T1 releases X on A, T2 reads A, then T1 aborts → T2 must abort too.

| Variant | Rule | Gives |
|---|---|---|
| **Basic 2PL** | Grow then shrink | Serializable |
| **Strict 2PL** ⭐ | Hold **X locks until commit/abort** | + cascadeless, recoverable |
| **Rigorous 2PL** | Hold **all locks** until commit/abort | + commit order = serial order |
| **Conservative 2PL** | Take all locks **before starting** | + no deadlocks, but low concurrency |

Most lock-based databases use **strict 2PL**.

> [!tip] 🎤 Say it like this
> "Reads take shared locks and writes take exclusive locks. Two-phase locking says a transaction first only acquires locks, then only releases them, which guarantees conflict serializability. In practice databases use strict 2PL, holding write locks until commit, so nobody reads uncommitted data and we avoid cascading aborts. The downside is deadlocks, so the DB needs detection."

---

## 4. Locks you use from SQL

```sql
BEGIN;
SELECT seats_left FROM flights WHERE id = 7 FOR UPDATE;  -- X lock on row
UPDATE flights SET seats_left = seats_left - 1 WHERE id = 7;
COMMIT;                                                  -- lock released
```

- `FOR UPDATE` — exclusive row lock (pessimistic locking).
- `FOR SHARE` — shared row lock.
- `FOR UPDATE SKIP LOCKED` — skip rows others hold; great for **job queues**.
- **Optimistic locking** (no DB lock): `UPDATE … WHERE id = 7 AND version = 3`; 0 rows changed → retry.

**✅ Pros (2PL)** — simple, proven, guaranteed serializable.
**❌ Cons** — readers block writers, **deadlocks**, lower throughput under contention. That's why many DBs prefer [MVCC](16-isolation-levels-and-mvcc.md) for reads.

> [!failure] Interview traps
> - Saying 2PL means "two-phase **commit**" — 2PC is for distributed transactions, totally different.
> - Thinking 2PL prevents deadlocks — it **causes** them (only conservative 2PL avoids them).
> - Forgetting phantoms: row locks can't lock rows that **don't exist yet** — need range/predicate (gap) locks.

---

## Summary

- **S** = shared read, **X** = exclusive write; only S+S is compatible.
- **2PL**: growing then shrinking → serializable.
- **Strict 2PL**: hold X locks to commit → no cascading aborts.
- Use `FOR UPDATE` for pessimistic, version column for optimistic.

> [!success] Key takeaway
> Locks make transactions wait their turn; 2PL makes the waiting order correct. The price is blocking and deadlocks.
