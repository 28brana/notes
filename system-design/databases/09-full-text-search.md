# 09 · Full-Text Search

> [!info] Topic info
> **Subject:** System Design · **Chapter:** Databases · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Databases](README.md) · [Interview questions →](questions/databases-qa.md)

> [!abstract] In one line
> A **full-text search engine** builds an **inverted index** (word → list of documents containing it) so it can find and **rank** matching text across billions of documents in milliseconds — Elasticsearch is the common choice.

---

## 1. The problem

Users want to search posts for "pizza recipe". In SQL that's `WHERE text LIKE '%pizza%'` — a **full table scan** over 1B posts, since a normal B-tree index can't help with a leading `%`.

And it still can't handle "Pizzas", typos like "piza", or rank the best results first.

---

## 2. The inverted index

Like the index at the back of a book: for each word, list where it appears.

| Term | Posting list (doc IDs) |
|---|---|
| `pizza` | 3, 7, 19, 204 |
| `recipe` | 7, 19, 88 |
| `best` | 7, 55 |

Search "pizza recipe" = intersect the two lists → docs **7, 19**. No scanning.

### Building it (analysis)

```mermaid
flowchart LR
    D[📝 Best Pizzas, ever!] --> T[✂️ Tokenize]
    T --> L[🔡 Lowercase]
    L --> S[🌱 Stem: pizzas to pizza]
    S --> X[🚫 Drop stop words]
    X --> I[📦 Inverted index]
```

The **same analysis** runs on the search query, so "PIZZAS" matches "pizza".

### Ranking

Results are scored by relevance, usually with **BM25**: a word scores higher if it's **frequent in this doc** but **rare across all docs**. Then you can boost by recency or likes.

---

## 3. How it fits in a system

```mermaid
flowchart LR
    App[🖥️ App] -->|writes| DB[🗄️ Postgres - source of truth]
    DB -->|CDC or events| Q[📬 Kafka]
    Q --> ES[🔍 Elasticsearch]
    App -->|search| ES
```

- Search engine is a **secondary index**, not the source of truth.
- Sync via [Change Data Capture](../big-data/02-change-data-capture.md) or events. Search is **eventually consistent** — a new post shows up after ~1 second.
- Elasticsearch shards the index and replicates each shard for scale and availability.

> [!tip] 🎤 Say it like this
> "For search I'd use Elasticsearch, which builds an inverted index mapping each term to the documents containing it, so lookups don't scan. Text is tokenized, lowercased, and stemmed, and results are ranked with BM25. Postgres stays the source of truth, and I'd stream changes into Elasticsearch through CDC, accepting that new posts appear in search a second or so later."

**✅ Use when:** keyword search, autocomplete, fuzzy/typo matching, faceted filters (e-commerce), log search (ELK stack).

**❌ Avoid when:** you need transactions or it would be the only copy of the data. For small apps, Postgres full-text search (`tsvector` + GIN index) may be enough.

> [!failure] Interview traps
> - Using `LIKE '%word%'` at scale.
> - Treating Elasticsearch as the primary database — it's not built for ACID or as the sole durable store.
> - Forgetting the sync path and its lag — the interviewer will ask "how does data get there?"

---

## Summary

- **Inverted index**: term → list of doc IDs.
- Analysis: tokenize → lowercase → stem → stop words.
- Rank with **BM25** (+ business boosts).
- Keep it as a **derived copy**, synced via CDC.

> [!success] Key takeaway
> Search = inverted index + ranking, fed from your main DB. Say "source of truth stays in Postgres".
