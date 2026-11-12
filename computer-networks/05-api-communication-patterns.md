# 05 · API Communication Patterns

> **Subject:** Computer Networks (for full stack developers)
> **Status:** ✅ done
> **Last reviewed:** 2026-11-13

[← Back to Computer Networks](README.md) · [Questions for this topic →](questions/05-api-communication-patterns-qa.md)

## In one line

There are many ways for two systems to talk — **request/response** (REST, GraphQL, gRPC), **real-time push** (WebSocket, SSE, polling) and **event callbacks** (webhooks). A senior dev picks the right one for the use case and makes it safe with **versioning, pagination and idempotency**.

---

## 1. REST APIs

> 🎤 **Say it like this:** "REST is an API style where everything is a resource with a URL, and we use HTTP methods to act on it — GET to read, POST to create, PUT/PATCH to update, DELETE to remove. It's stateless, uses standard status codes and is cache-friendly. It's simple and works everywhere, which is why it's the default for most public APIs."

**Principles:** resources (nouns), HTTP methods (verbs), stateless, standard status codes, cacheable, uniform interface.

**Good URL design:**

| ✅ Good | ❌ Bad |
|---|---|
| `GET /users` | `GET /getAllUsers` |
| `GET /users/42` | `GET /user?id=42&action=get` |
| `POST /users` | `POST /createUser` |
| `PATCH /users/42` | `POST /updateUser` |
| `GET /users/42/orders` | `GET /ordersOfUser/42` |
| `GET /orders?status=paid&sort=-createdAt` | |

- Plural nouns, lowercase, hyphens (`/order-items`).
- Nesting max 1–2 levels.
- Actions that don't fit CRUD: `POST /orders/42/cancel` is fine.
- Return correct codes: `201` created, `204` no content, `400` validation, `401/403`, `404`, `409` conflict, `422`, `429`. See [02 · HTTP and HTTPS](02-http-and-https.md).
- Document it with **OpenAPI / Swagger** — gives docs, client SDK generation and request validation.

---

## 2. GraphQL

> 🎤 **Say it like this:** "GraphQL has one endpoint and the client sends a query describing exactly which fields it wants. It solves over-fetching and under-fetching, which is great for mobile and complex UIs. The trade-offs are harder HTTP caching, the N+1 query problem, and needing query depth/cost limits for security."

```graphql
query {
  user(id: 42) {
    name
    orders(last: 3) { id total }
  }
}
```

- **Over-fetching:** REST returns 30 fields, you need 2.
- **Under-fetching:** you need user + orders + reviews → 3 REST calls. GraphQL = 1 call.
- **N+1 problem:** fetching 50 users, then each user's orders resolver runs 1 query → 1 + 50 queries.
  **Fix: DataLoader** — collects all IDs in one tick and runs **one batched query** (`WHERE user_id IN (...)`), plus per-request caching.
- Usually `POST /graphql` → no CDN caching by default (use persisted queries / client cache like Apollo).
- Always returns `200` even for errors (errors in `errors` array) — monitoring must check that.
- Protect with **query depth limit, cost analysis, disable introspection in prod**.

---

## 3. gRPC (conceptual)

> 🎤 **Say it like this:** "gRPC is a high-performance RPC framework from Google. You define services in a `.proto` file, it generates typed clients and servers, sends data as binary Protocol Buffers over HTTP/2, and supports streaming in both directions. It's great for internal service-to-service communication, but not directly usable from browsers without gRPC-Web."

```proto
service UserService {
  rpc GetUser (GetUserRequest) returns (User);
}
```

### REST vs GraphQL vs gRPC

| | REST | GraphQL | gRPC |
|---|---|---|---|
| Endpoint | Many (per resource) | One `/graphql` | Methods on a service |
| Data format | JSON | JSON | Protobuf (binary) |
| Transport | HTTP/1.1 or 2 | HTTP (usually POST) | HTTP/2 |
| Client decides fields | ❌ | ✅ | ❌ (fixed by proto) |
| HTTP caching | ✅ Easy | ❌ Harder | ❌ |
| Typed contract | OpenAPI (optional) | Schema (built-in) | `.proto` (strict) |
| Browser support | ✅ Native | ✅ Native | ⚠️ needs gRPC-Web |
| Streaming | Limited | Subscriptions | ✅ Bi-directional |
| Best for | Public APIs, CRUD | Complex UIs, mobile, BFF | Internal microservices, low latency |

---

## 4. Real-time: polling, long polling, SSE, WebSockets

### Short polling
Client asks every N seconds: "anything new?" Simple but wasteful; most responses are empty.

### Long polling
> 🎤 "Client sends a request and the server **holds it open** until there's new data or a timeout, then responds. The client immediately sends the next request. Feels real-time, works over plain HTTP, but each message costs a new request."

### Server-Sent Events (SSE)
> 🎤 "SSE is a one-way stream from server to client over a normal HTTP response with `Content-Type: text/event-stream`. The browser's `EventSource` auto-reconnects and resumes using `Last-Event-ID`. Perfect for notifications, live feeds, and streaming AI responses."

```js
// server (Express)
app.get('/events', (req, res) => {
  res.set({ 'Content-Type': 'text/event-stream', 'Cache-Control': 'no-cache', Connection: 'keep-alive' });
  const t = setInterval(() => res.write(`data: ${JSON.stringify({ time: Date.now() })}\n\n`), 1000);
  req.on('close', () => clearInterval(t));
});
// browser
new EventSource('/events').onmessage = (e) => console.log(JSON.parse(e.data));
```

### WebSockets
> 🎤 "WebSocket is a full-duplex, persistent connection — both sides can send any time. It starts as an HTTP request with `Upgrade: websocket`, the server replies `101 Switching Protocols`, and then the same TCP connection is used for WebSocket frames. Used for chat, multiplayer games, collaborative editing."

```
GET /chat HTTP/1.1
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZQ==

HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

**Scaling WebSockets (senior topic):**
- Connections are **stateful** — user A on server 1, user B on server 2. How does A's message reach B?
  → **Redis Pub/Sub** (or Kafka/NATS) between servers; Socket.IO has a Redis adapter.
- Load balancer must support upgrade + long idle timeout; often **sticky sessions** (required for Socket.IO's polling fallback).
- Use **heartbeats (ping/pong)** to detect dead connections, and reconnect with backoff on the client.
- Auth: send token on connect (cookie or first message), not in the URL if avoidable (URLs get logged).

### Comparison

| | Short polling | Long polling | SSE | WebSocket |
|---|---|---|---|---|
| Direction | Client → pull | Client → pull | Server → client | Both ways |
| Connection | New each time | New per message | 1 long HTTP | 1 persistent TCP |
| Latency | High (interval) | Low | Low | Lowest |
| Server cost | High (wasted calls) | Medium | Low | Low–Medium (stateful) |
| Auto reconnect | n/a | Manual | ✅ Built in | Manual |
| Binary data | ✅ | ✅ | ❌ Text only | ✅ |
| Works through proxies | ✅ | ✅ | ✅ (plain HTTP) | Mostly, needs upgrade support |

**Which would you choose?**

| Use case | Choice | Why |
|---|---|---|
| Chat app | **WebSocket** | Both sides send often, low latency |
| Notifications | **SSE** | Server → client only, simple, auto-reconnect |
| Live cricket/football score | **SSE** | One-way updates to many users |
| Stock ticker / trading | **WebSocket** | High-frequency, client also subscribes/unsubscribes |
| AI chat response streaming | **SSE** (or streamed fetch) | One-way token stream |
| Status of a job every 30s, simple admin page | **Short polling** | Simplest, low frequency |
| Old infra that blocks WebSockets | **Long polling** | Plain HTTP fallback |

---

## 5. Webhooks

> 🎤 **Say it like this:** "A webhook is a reverse API — instead of me polling Stripe for payment status, Stripe sends an HTTP POST to my URL when the event happens. On my side I verify the signature with HMAC, return 2xx quickly, process the event asynchronously, and make the handler idempotent because the provider will retry and may send duplicates."

```
Stripe ──POST /webhooks/stripe {event: payment.succeeded}──▶ Your API
       ◀──────────────── 200 OK (fast) ─────────────────────
                                  │ push to queue → process later
```

**Checklist:**
1. **Verify signature** — provider signs the raw body with a shared secret (HMAC-SHA256); you recompute and compare (constant-time). Also check timestamp to block replays.
2. **Respond fast (2xx)** — do heavy work in a queue; slow responses cause timeouts and retries.
3. **Idempotent** — store `event.id`; if seen already, skip. Retries = duplicates.
4. **Don't trust order** — events can arrive out of order; check current state.

```js
app.post('/webhooks/stripe', express.raw({ type: 'application/json' }), (req, res) => {
  const expected = crypto.createHmac('sha256', SECRET).update(req.body).digest('hex');
  const ok = crypto.timingSafeEqual(Buffer.from(expected), Buffer.from(req.get('X-Signature') || ''));
  if (!ok) return res.sendStatus(401);
  queue.add(JSON.parse(req.body));
  res.sendStatus(200);
});
```
> 💡 Use the **raw body** for signature check — `express.json()` re-serialising changes the bytes.

---

## 6. File uploads and multipart/form-data

> 🎤 **Say it like this:** "For small uploads I send `multipart/form-data` — the body is split into parts by a boundary, each part with its own headers, so files and text fields go together. On the server I use multer or busboy to stream it. For large files I don't pass them through my server at all — the backend gives a presigned S3 URL and the browser uploads directly to S3."

```
Content-Type: multipart/form-data; boundary=----abc
------abc
Content-Disposition: form-data; name="title"

My photo
------abc
Content-Disposition: form-data; name="file"; filename="cat.png"
Content-Type: image/png

<binary bytes>
------abc--
```

```js
const fd = new FormData();
fd.append('title', 'My photo');
fd.append('file', fileInput.files[0]);
fetch('/upload', { method: 'POST', body: fd }); // don't set Content-Type manually — browser adds boundary
```

**Presigned URL flow (preferred for big files):**
```
1. Browser → API: "I want to upload cat.png (5 MB)"
2. API → validates, returns presigned PUT URL (valid 5 min)
3. Browser → S3: PUT file directly
4. Browser → API: "done, key = uploads/123.png"  (or S3 event notifies backend)
```
Benefits: server bandwidth/memory not used, scales easily.

**Large files: chunked / resumable upload** — split into parts (e.g. 5–10 MB), upload in parallel, retry only failed parts. S3 **Multipart Upload** or the **tus** protocol. Validate type and size on the server, never trust the extension.

---

## 7. Streaming APIs

- Server sends the response **in pieces** as it's ready: big CSV exports, logs, AI tokens.
- Techniques: chunked HTTP response, SSE, NDJSON (one JSON per line), gRPC streams.
- Client reads with `response.body.getReader()` in fetch.
- Node side → [04 · Streaming responses](04-backend-networking-nodejs.md#12-streaming-responses).

---

## 8. API versioning

> 🎤 **Say it like this:** "Versioning lets me make breaking changes without breaking existing clients — especially mobile apps that can't be force-updated. Most common is a URL version like `/v1/users`. I avoid breaking changes when possible by only adding fields, and when I do bump a version I deprecate the old one with a timeline."

| Strategy | Example | Notes |
|---|---|---|
| **URL path** | `/api/v1/users` | Most common, clear, easy to route & cache |
| Query param | `/users?version=2` | Easy but messy |
| Header | `Accept-Version: 2` / `Accept: application/vnd.app.v2+json` | Clean URLs, harder to test in browser |
| Date-based | `Stripe-Version: 2024-06-20` | Stripe style, fine-grained |

- **Non-breaking:** adding a field or a new endpoint. **Breaking:** removing/renaming fields, changing types, changing meaning.
- Signal deprecation with `Deprecation` / `Sunset` headers and docs.

---

## 9. Pagination

> 🎤 **Say it like this:** "Offset pagination uses LIMIT and OFFSET — simple, supports jumping to page 7, but gets slow on big tables because the DB still scans skipped rows, and items shift when new data is inserted. Cursor pagination uses the last seen id or timestamp — `WHERE id < cursor LIMIT 20` — so it's fast with an index and stable for infinite scroll. I use cursor for feeds and large data, offset for small admin tables."

| | Offset | Cursor (keyset) |
|---|---|---|
| Request | `?page=3&limit=20` | `?cursor=eyJpZCI6MTIzfQ&limit=20` |
| SQL | `LIMIT 20 OFFSET 40` | `WHERE id < 123 ORDER BY id DESC LIMIT 20` |
| Speed on big tables | ❌ Slower as offset grows | ✅ Constant (uses index) |
| New rows inserted | Duplicates / skipped items | Stable |
| Jump to page N | ✅ | ❌ Only next/prev |
| Use for | Admin tables, small data | Feeds, infinite scroll, APIs |

Response shape: `{ data: [...], nextCursor: "...", hasMore: true }`. Keep the cursor **opaque** (base64) so you can change it later.

---

## 10. Idempotency keys

> 🎤 **Say it like this:** "Idempotent means doing the same request twice has the same effect as once. GET, PUT and DELETE are idempotent, POST is not. For things like payments, the client generates a unique Idempotency-Key and sends it with the POST. The server stores the key with the result; if a retry comes with the same key, it returns the saved response instead of charging again."

```
Client: POST /payments   Idempotency-Key: 7f3a-...   {amount: 500}
Server: key new? → lock key → charge → save {key, status, response} → 201
        ── network drops, client never sees response ──
Client: retry same request, same key
Server: key exists → return saved 201 response (no second charge)
        key exists but still processing → 409 / wait
```

- Store keys in Redis/DB with a TTL (e.g. 24h); also save a hash of the body — same key + different body → `422`.
- Use a DB **unique constraint** on the key to avoid race conditions between two parallel retries.
- Needed whenever there are retries: payments, orders, emails, webhook consumers.

---

## ⚡ Quick revision

- REST = resources + HTTP verbs, stateless, cacheable; plural nouns, correct status codes, OpenAPI docs.
- GraphQL = one endpoint, client picks fields; watch N+1 (→ DataLoader), caching, query limits.
- gRPC = protobuf over HTTP/2, typed, streaming; best for internal services.
- Polling (simple, wasteful) → long polling → SSE (server→client, auto-reconnect) → WebSocket (two-way).
- WebSocket = HTTP `Upgrade` → `101`; scale with Redis pub/sub + sticky sessions + heartbeats.
- Webhooks = verify HMAC on raw body, 2xx fast, queue it, dedupe by event id.
- Uploads: multipart for small; presigned S3 URL for large; multipart/resumable for huge.
- Versioning: `/v1/` most common; only additive changes when possible.
- Pagination: offset = simple but slow/unstable; cursor = fast + stable.
- Idempotency-Key makes POST retries safe (payments).
