# 07 · Document Databases

> [!info] Topic info
> **Subject:** System Design · **Chapter:** Databases · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Databases](README.md) · [Interview questions →](questions/databases-qa.md)

> [!abstract] In one line
> A **document database** stores each record as a self-contained **JSON-like document** with a **flexible schema**, so data that's read together is **stored together** — MongoDB is the classic example.

---

## 1. The problem

A user profile on our app has a name, bio, a list of links, privacy settings, and notification preferences. In SQL that's 4–5 tables and joins just to render one profile page.

And every week product adds a new optional field. Each one means a schema migration.

---

## 2. How it works

One document holds the whole thing:

```json
{
  "_id": "u42",
  "name": "Asha",
  "links": ["github.com/asha", "asha.dev"],
  "settings": { "private": false, "notify": { "email": true, "push": false } }
}
```

- Documents live in **collections** (like tables). Documents in one collection can have **different fields**.
- Read the profile = **one lookup**, no joins.
- You can index nested fields (`settings.private`) and query them.
- Scales horizontally by **sharding on a key** like `_id` or `user_id`.

### Embed vs reference

| | Embed (nest inside) | Reference (store ID) |
|---|---|---|
| **Use when** | Data is owned by the parent and read together | Data is shared or grows without limit |
| **Example** | User's settings, address | A post's comments (could be 100K) |
| **Pro** | One read | No huge documents, no duplicates |

> [!warning] Unbounded arrays
> Don't embed things that grow forever (comments, followers). Documents have size limits (16 MB in MongoDB) and get slow to rewrite.

---

## 3. When to use / when not

**✅ Use when**
- Records are **self-contained** and read as a whole: profiles, product catalogs, CMS content, game state.
- Schema **changes often** or varies per record.
- Mostly one-to-many relationships, few cross-record joins.

**❌ Avoid when**
- Data is highly **connected** (many-to-many: follows, likes across users). You'll end up doing joins in app code.
- You need complex multi-record transactions everywhere (MongoDB supports them now, but they're slower and not its sweet spot).
- Heavy ad-hoc analytics across collections.

> [!tip] 🎤 Say it like this
> "A document DB stores each entity as one JSON document, so a profile or product page is a single read with no joins, and the schema can evolve freely. It fits self-contained data like catalogs or profiles. I'd avoid it for heavily relational data like a social graph, where I'd end up doing joins in application code."

> [!note] Postgres can do this too
> Postgres **JSONB** columns give document-style storage with indexes, inside a relational DB. Often a good middle ground.

> [!failure] Interview traps
> - "Schemaless" means **no schema** — false. The schema moves into app code, and old documents with old shapes stay around.
> - Copying the same data into many documents, then forgetting to update all copies.

---

## Summary

| | Document DB |
|---|---|
| **Model** | JSON documents in collections |
| **Best for** | Self-contained, nested, evolving records |
| **Weak at** | Many-to-many relations, cross-document joins |
| **Examples** | MongoDB, Couchbase, Firestore, DynamoDB (doc mode) |

> [!success] Key takeaway
> Store together what you **read** together. If your data is a graph of relations, stay relational.
