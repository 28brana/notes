# 04 · Data Formats

> [!info] Topic info
> **Subject:** System Design · **Chapter:** API Fundamentals · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to API Fundamentals](README.md) · [Interview questions →](questions/api-fundamentals-qa.md)

> [!abstract] In one line
> A **data format** decides how data is turned into bytes on the wire (**serialization**) — **text** formats like **JSON** are easy to read, **binary** formats like **Protobuf** are smaller and faster.

---

## 1. The problem

Our feed service sends a post object to 10 other services, **50,000 times per second**. As JSON, each post is ~1 KB. That's **50 MB/s** of mostly repeated field names like `"created_at"`.

Every service also spends CPU **parsing** text. At this scale, the format itself becomes a cost.

---

## 2. Serialization in one picture

**Serialization** = object in memory → bytes. **Deserialization** = bytes → object again. Both sides must agree on the format.

```mermaid
flowchart LR
    A[⚙️ Feed service object] -- serialize --> B[📦 Bytes on the wire]
    B -- deserialize --> C[⚙️ Notification service object]
```

---

## 3. Common formats

| Format | Type | Schema | Size / speed | Used for |
|---|---|---|---|---|
| **JSON** | Text | Optional | Medium | REST, GraphQL, browsers — the default |
| **XML** | Text | Optional (XSD) | Large, slow | SOAP, old enterprise systems |
| **Protobuf** | Binary | **Required** (`.proto`) | Small, very fast | gRPC, internal services |
| **Avro** | Binary | **Required** | Small, fast | Kafka, data pipelines |
| **MessagePack** | Binary | None | Small | "Binary JSON", caches |

Same user, roughly:

```
JSON:      {"id":42,"name":"Bharat","verified":true}   → ~40 bytes
Protobuf:  field 1 = 42, field 2 = "Bharat", field 3 = 1 → ~12 bytes
```

Protobuf sends **field numbers** instead of names — that's where the 3–10× saving comes from.

> [!tip] 🎤 Say it like this
> "For public and browser-facing APIs I use JSON — it's human-readable, every language supports it, and it's easy to debug. For high-volume internal traffic I use a binary format like Protobuf: it's several times smaller and faster to parse, and the schema gives type safety. For event streams like Kafka, Avro is common because of its schema evolution support."

---

## 4. Text vs binary

**✅ JSON (text) pros**
- Human-readable — easy to debug with curl and logs
- Works everywhere, native in browsers
- No schema needed to start

**❌ JSON cons**
- Bigger payloads, slower parsing
- No types enforced (`"42"` vs `42` bugs)

**✅ Protobuf / Avro (binary) pros**
- 3–10× smaller, much faster to parse
- Strict schema = typed contract between teams
- Built-in **schema evolution** — add fields without breaking old clients

**❌ Binary cons**
- Not human-readable — need tools to inspect
- Both sides need the schema / generated code

> [!important] Schema evolution rule
> Only **add** new optional fields. Never **reuse or change** a field number/type. That's how old and new services keep talking during a rolling deploy.

> [!failure] Interview traps
> - Saying "binary is always better" — for a public API, readability and tooling usually win.
> - Forgetting **compression** (gzip / brotli) — it shrinks JSON a lot over HTTP.
> - Using floats for money in JSON — use integers (paise/cents) or strings.

---

## Summary

- **JSON** → default for public/web APIs; readable, universal.
- **Protobuf** → internal, high-QPS, gRPC; small, fast, typed.
- **Avro** → event streams with evolving schemas.
- Binary saves bandwidth and CPU; text saves developer time.

> [!success] Key takeaway
> JSON at the edge, binary inside — and with binary, **evolve schemas by adding fields only**.
