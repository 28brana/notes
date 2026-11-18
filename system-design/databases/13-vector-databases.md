# 13 · Vector Databases

> [!info] Topic info
> **Subject:** System Design · **Chapter:** Databases · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Databases](README.md) · [Interview questions →](questions/databases-qa.md)

> [!abstract] In one line
> A **vector database** stores **embeddings** — lists of numbers that capture meaning — and finds the **most similar** ones fast using **approximate nearest neighbor (ANN)** indexes like **HNSW** — e.g. pgvector, Pinecone, Weaviate.

---

## 1. The problem

A user searches "cheap places to eat in Rome". The best post says "budget trattorias near the Colosseum". Not one word matches, so [keyword search](09-full-text-search.md) misses it.

We want to search by **meaning**, not exact words. Same for "posts similar to this one" and for feeding an LLM relevant context (RAG).

---

## 2. Embeddings

An **embedding model** turns text (or an image) into a vector, e.g. 768 numbers. Similar meaning → vectors **close together** in that space.

- "cheap food Rome" and "budget trattorias Colosseum" → close.
- "cheap food Rome" and "Python tutorial" → far apart.

**Closeness** is measured with **cosine similarity** (angle between vectors) or dot product / Euclidean distance.

```mermaid
flowchart LR
    T[📝 Post text] --> E[🧠 Embedding model]
    E --> V[🔢 Vector 768 numbers]
    V --> DB[🧭 Vector DB + ANN index]
    Q[🔍 Query text] --> E2[🧠 Same model]
    E2 --> DB
    DB --> R[📋 Top 10 nearest]
```

---

## 3. Finding neighbours fast — ANN

**Exact search** compares the query to every vector: 100M vectors × 768 dims = way too slow per query.

**ANN** trades a little accuracy (maybe 95–99% **recall**, i.e. finds 95–99 of the true top 100) for **huge speed** (ms instead of seconds).

### HNSW (intuition)

**HNSW** (Hierarchical Navigable Small World) is a **multi-layer graph**, like a highway system:

- **Top layer**: few nodes, long links — like highways between cities.
- **Lower layers**: more nodes, short links — like local streets.
- Search starts at the top, greedily jumps toward the query, drops a layer, repeats, until it reaches the closest neighbours at the bottom.

✅ Very fast, high recall. ❌ Uses lots of RAM, and building the index is slow.

> [!note] Other index: IVF
> **IVF** clusters vectors into buckets and only searches the few closest buckets. Uses less memory than HNSW, a bit lower recall.

---

## 4. Choosing one

| | pgvector | Pinecone / Weaviate / Milvus |
|---|---|---|
| **What** | Postgres extension | Dedicated vector DB |
| **Good for** | Up to ~millions of vectors, already on Postgres, need joins/filters | Hundreds of millions+, managed scaling |
| **Bonus** | Vectors next to your normal data, one system | Built-in sharding, hybrid search features |

> [!tip] 🎤 Say it like this
> "We convert each post into an embedding with a model, store the vectors, and at query time embed the query and find the nearest vectors by cosine similarity. Exact search is too slow at scale, so we use an approximate index like HNSW — a layered graph that narrows in on neighbours quickly — trading a small bit of recall for millisecond latency. I'd start with pgvector and move to a dedicated vector DB only at very large scale."

> [!failure] Interview traps
> - Mixing embedding models — query and data must use the **same model**; changing models means re-embedding everything.
> - Forgetting **filters** ("similar posts, but only public ones in English") — filtering hurts ANN recall if not supported natively.
> - Thinking vector search replaces keyword search — for names, codes, and exact terms, keyword wins. Many systems use **hybrid search** (both, merged).

---

## Summary

- **Embedding** = meaning as numbers; similar meaning → close vectors.
- **ANN** (HNSW, IVF) = fast, approximate nearest neighbour search.
- Uses: semantic search, recommendations, RAG for LLMs, duplicate detection.

> [!success] Key takeaway
> Vector DB = "find things **like** this". Embed → index with HNSW → query top-k by cosine similarity.
