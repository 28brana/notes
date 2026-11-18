# 11 · MinHash

> [!info] Topic info
> **Subject:** System Design · **Chapter:** Data Structures for Scale · **Priority:** 💤 Low · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Data Structures for Scale](README.md) · [Interview questions →](questions/data-structures-qa.md)

> [!abstract] In one line
> **MinHash** shrinks a big document into a small **signature** (e.g. 128 numbers) so you can quickly estimate how **similar** two documents are — the basis of **near-duplicate detection**.

---

## 1. The problem

Spammers on our app post the same text with tiny changes: an extra emoji, a different link. Exact hashing (MD5) misses them — **one character changes the whole hash**.

We want "these two posts are **90% the same**" across **millions** of posts, without comparing every pair word by word.

---

## 2. How it works

1. **Shingles:** break each post into overlapping chunks, e.g. every 3 words. Now each post is a **set**.
2. **Similarity = Jaccard:** shared shingles ÷ total distinct shingles. 1.0 = identical, 0 = nothing in common.
3. **MinHash:** apply a hash function to every shingle and keep only the **smallest** value. Do this with 128 different hash functions → a **128-number signature**.

The magic: the chance two posts have the **same min-hash** equals their **Jaccard similarity**. So if 115 of 128 values match → about **90% similar**.

```mermaid
flowchart LR
    P[📝 Post text] --> S[⚙️ Shingles set]
    S --> M[⚙️ 128 min hashes]
    M --> L[📦 LSH buckets]
    L --> C[✅ Candidate near-duplicates]
```

### LSH — avoid comparing every pair

**Locality-Sensitive Hashing (LSH)** splits the signature into bands and hashes each band into buckets. Similar posts likely land in the **same bucket** in at least one band. Only compare posts that share a bucket.

> [!tip] 🎤 Say it like this
> "For near-duplicate detection I'd break each document into shingles, compute a MinHash signature of around 128 values, and use LSH banding so only likely-similar documents get compared. The fraction of matching MinHash values estimates the Jaccard similarity."

> [!failure] Interview traps
> - Using exact hashes (MD5/SHA) for **near**-duplicates — they only catch **exact** copies.
> - Comparing all pairs of signatures — that's O(n²). Mention **LSH** to get candidates.

---

## Summary

- Doc → shingles → MinHash signature → LSH buckets → compare candidates.
- Matching signature fraction ≈ **Jaccard similarity**.
- Uses: dedup web pages in crawlers, spam / plagiarism detection, similar-item search.

> [!success] Key takeaway
> "Detect **near**-duplicate content at scale" → **MinHash + LSH**.
