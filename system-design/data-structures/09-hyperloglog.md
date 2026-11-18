# 09 · HyperLogLog

> [!info] Topic info
> **Subject:** System Design · **Chapter:** Data Structures for Scale · **Priority:** 💤 Low · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Data Structures for Scale](README.md) · [Interview questions →](questions/data-structures-qa.md)

> [!abstract] In one line
> **HyperLogLog (HLL)** estimates the number of **unique** items in a huge set using a **fixed ~12 KB** of memory with about **0.81% error** — perfect for "unique visitors" counts.

---

## 1. The problem

We want "**unique viewers**" on every post. A viral post gets **500 million** views from **200 million** distinct users.

Exact counting means storing every user ID in a set: 200M × 8 bytes ≈ **1.6 GB — for one post**. Multiply by millions of posts. Impossible. Nobody needs the exact number; "~200M" is fine.

---

## 2. How it works (intuition)

Hash each user ID into random-looking bits. Now look at **how many leading zeros** it starts with.

- 1 leading zero: happens to 1 in 2 hashes.
- 10 leading zeros: about 1 in 1,024 hashes.

So if the **longest run of zeros** you've ever seen is 10, you've probably seen around **1,000 distinct** items. Duplicates hash the same, so they don't change anything.

One estimate is noisy, so HLL splits hashes into many **buckets** (Redis uses 16,384), tracks the max per bucket, and **averages** them.

```mermaid
flowchart LR
    V[👥 View events] --> H[⚙️ Hash user id]
    H --> B[📦 Pick bucket, count leading zeros]
    B --> M[⚡ Keep max per bucket]
    M --> E[✅ Estimated unique count]
```

> [!tip] 🎤 Say it like this
> "For unique counts at scale I'd use HyperLogLog. It hashes each item and tracks the longest run of leading zeros per bucket — rare patterns mean lots of distinct items. In Redis it's PFADD and PFCOUNT, about 12 KB per counter with 0.81% standard error, no matter if it's a thousand or a billion users."

---

## 3. In practice

- **Redis:** `PFADD post:42 user123`, `PFCOUNT post:42`, `PFMERGE` to combine.
- **Mergeable:** daily HLLs can be merged into weekly/monthly — **no double counting** of the same user. Exact counters can't do that.
- Also in BigQuery / Presto `APPROX_COUNT_DISTINCT`.

**✅ Pros** — fixed tiny memory, mergeable across days or servers.
**❌ Cons** — approximate; can't list the users; can't answer "did user X view?" (that's a [Bloom filter](05-bloom-filters.md)).

> [!failure] Interview traps
> - Using it for **billing or payments** — those need exact counts.
> - Adding up daily unique counts to get weekly — wrong, users repeat. **Merge the HLLs** instead.

---

## Summary

- Counts **distinct** items with ~12 KB and ~0.81% error.
- Trick: longest run of leading zeros in hashes, averaged across buckets.
- Redis `PFADD` / `PFCOUNT` / `PFMERGE`. Mergeable.

> [!success] Key takeaway
> "Unique visitors / DAU at huge scale" → **HyperLogLog**.
