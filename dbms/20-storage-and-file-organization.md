# 20 · Storage & File Organization

> [!info] Topic info
> **Subject:** DBMS · **Part:** Transactions & Internals · **Priority:** 💤 Low · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to DBMS](README.md) · [Interview questions →](questions/transactions-and-internals-qa.md)

> [!abstract] In one line
> Databases store rows inside fixed-size **pages** (~8–16 KB) in files on disk, cache hot pages in a **buffer pool**, and organize records as a **heap**, **sorted** file, or **hashed** file.

---

## 1. What it is

Disk reads happen in blocks, and a random disk read costs ~100× a RAM read. So the DB never reads "one row" — it reads the **page** that contains it.

Everything about storage is designed to **read fewer pages**.

---

## 2. How it works

```mermaid
flowchart LR
    Q[⚙️ Query executor] --> BP[⚡ Buffer pool in RAM]
    BP -- page miss --> F[🗄️ Data file on disk]
    F --> P[📦 Page 8 KB: header, slots, rows]
```

- **Page layout (slotted page):** a header, a slot array pointing to each row, and rows filling from the end. Rows can move inside the page without changing their ID (page no + slot no).
- **Buffer pool:** RAM cache of pages (Postgres `shared_buffers`, InnoDB buffer pool). Evicts with an LRU-like policy. Changed pages are **dirty** and flushed later — safe thanks to the [WAL](18-recovery-and-wal.md).
- **Big values** (long text, JSON) go to overflow pages (Postgres **TOAST**).

---

## 3. File organizations

| Organization | How rows are placed | Good for | Bad for |
|---|---|---|---|
| **Heap** | Anywhere there's space | Fast inserts | Search needs an index or full scan |
| **Sorted / sequential** | Ordered by a key | Range scans | Inserts must shift or overflow |
| **Hashed** | Bucket = hash of key | Equality lookups | Ranges |
| **Clustered (index-organized)** | Inside the PK B+ tree | PK lookups and ranges | Random-key inserts cause splits |

Postgres tables are heaps; InnoDB tables are clustered on the PK — see [Indexing & B+ Trees](19-indexing-b-plus-trees.md).

**Row vs column storage:** OLTP DBs store whole rows together (fast for "get this order"). Analytics DBs (Redshift, ClickHouse) store each **column** together — compresses well and `SUM(amount)` reads only one column.

> [!tip] 🎤 Say it like this
> "Data lives in fixed-size pages, and the database caches hot pages in a buffer pool, so performance is mostly about how many pages a query touches. Rows can be stored as an unordered heap like Postgres, or clustered by primary key like InnoDB. For analytics, columnar storage wins because queries read only the columns they need."

> [!failure] Interview traps
> - Thinking the DB reads single rows from disk — it reads whole pages.
> - Assuming a bigger buffer pool fixes everything — a bad query still scans millions of pages.

---

## Summary

- Unit of I/O = **page**; RAM cache = **buffer pool**.
- Heap (fast insert), sorted (ranges), hashed (equality), clustered (PK order).
- Row store for OLTP, column store for analytics.

> [!success] Key takeaway
> Fewer pages touched = faster query. Indexes, clustering and columnar storage are all ways to touch fewer pages.
