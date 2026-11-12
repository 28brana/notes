# Q&A · API Communication Patterns

[← Back to Computer Networks](../README.md) · [Read the notes →](../05-api-communication-patterns.md)

> Short answers you can say out loud. 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

### 🟢 Q1. What is REST?
An API style where everything is a resource with a URL and HTTP methods act on it (GET, POST, PUT, PATCH, DELETE). It's stateless, uses standard status codes and is cacheable.

### 🟢 Q2. What makes a good REST URL?
Plural nouns, no verbs, lowercase with hyphens, shallow nesting: `GET /users/42/orders?status=paid`. Not `/getUserOrders?id=42`. Actions that aren't CRUD can be `POST /orders/42/cancel`.

### 🟡 Q3. REST vs GraphQL?
REST has many endpoints with fixed responses and easy HTTP caching. GraphQL has one endpoint where the client asks for exactly the fields it needs — no over/under-fetching — but caching is harder and you must handle N+1 and query cost limits.

### 🟡 Q4. What is over-fetching and under-fetching?
Over-fetching: the API returns more fields than you need. Under-fetching: one call doesn't give enough, so you make several calls. GraphQL solves both by letting the client shape the response.

### 🔴 Q5. What is the N+1 problem in GraphQL and how do you fix it?
Fetching a list of N items, then a resolver runs one extra query per item → 1 + N queries. Fix with DataLoader: it batches all requested IDs in one tick into a single `WHERE id IN (...)` query and caches per request.

### 🟡 Q6. What is gRPC and when would you use it?
An RPC framework using Protocol Buffers (binary) over HTTP/2, with generated typed clients and streaming. Best for internal microservice communication where speed and strict contracts matter. Browsers need gRPC-Web.

### 🟢 Q7. What is a WebSocket?
A persistent, full-duplex connection where both client and server can send messages any time. Used for chat, games, live collaboration.

### 🟡 Q8. How does a WebSocket connection start?
As a normal HTTP GET with `Upgrade: websocket` and `Connection: Upgrade` headers. Server replies `101 Switching Protocols`, and the same TCP connection is then used for WebSocket frames.

### 🟡 Q9. SSE vs WebSocket?
SSE is one-way (server → client) over plain HTTP, text only, with built-in auto-reconnect. WebSocket is two-way, supports binary, but needs more setup and reconnect handling. If the client only listens, choose SSE.

### 🟢 Q10. Short polling vs long polling?
Short polling asks every few seconds — simple but mostly empty responses. Long polling sends a request that the server holds open until data arrives or it times out, then the client immediately re-requests — near real-time over plain HTTP.

### 🟡 Q11. Which would you use for chat, notifications, live scores and a stock ticker?
Chat → WebSocket (two-way). Notifications → SSE. Live scores → SSE (one-way broadcast). Stock ticker → WebSocket (high frequency, client subscribes/unsubscribes).

### 🔴 Q12. How do you scale WebSockets across multiple servers?
Connections are stateful, so users on different servers can't see each other's messages directly. Use Redis Pub/Sub (or Kafka) to broadcast between servers, configure the load balancer for upgrade + long idle timeout (sticky sessions if needed), and add ping/pong heartbeats and client reconnect with backoff.

### 🟢 Q13. What is a webhook?
A reverse API: the provider sends an HTTP POST to your URL when an event happens (e.g. Stripe payment succeeded), instead of you polling them.

### 🔴 Q14. How do you build a reliable webhook receiver?
Verify the HMAC signature on the raw body (and timestamp), return 2xx quickly, push the event to a queue for processing, and make it idempotent by storing the event ID — providers retry and can send duplicates or out of order.

### 🟢 Q15. What is multipart/form-data?
A content type for sending files and form fields in one request. The body is split into parts by a boundary string, each with its own headers. With `FormData` in fetch, let the browser set the Content-Type so the boundary is included.

### 🟡 Q16. How would you handle large file uploads?
Don't pass them through the API server. Backend returns a presigned S3 URL, the browser uploads directly to S3, then notifies the backend. For very large files, use multipart/chunked upload so parts upload in parallel and only failed parts retry.

### 🟡 Q17. How do you version an API?
Most common is URL versioning `/api/v1/...`. Alternatives: header (`Accept-Version`) or date-based (Stripe). Prefer additive, non-breaking changes; when breaking, release a new version and deprecate the old one with a timeline.

### 🟡 Q18. Offset vs cursor pagination?
Offset (`LIMIT 20 OFFSET 40`) is simple and allows page jumps, but slows down on large offsets and shows duplicates when new rows are added. Cursor (`WHERE id < lastId LIMIT 20`) uses an index, stays fast and stable — best for feeds and infinite scroll.

### 🟢 Q19. What does idempotent mean? Which methods are idempotent?
Repeating the request gives the same result as doing it once. GET, PUT, DELETE (and HEAD, OPTIONS) are idempotent; POST is not; PATCH usually isn't guaranteed.

### 🔴 Q20. A user clicks "Pay" and the network drops. The app retries. How do you prevent a double charge?
Idempotency key: the client sends a unique `Idempotency-Key` header with the POST. The server stores the key (unique constraint) with the result. A retry with the same key returns the saved response instead of charging again; a key still in progress returns 409.

### 🟢 Q21. What is OpenAPI / Swagger?
A standard spec (YAML/JSON) to describe REST APIs — endpoints, params, schemas, responses. Swagger UI turns it into interactive docs, and tools generate client SDKs and validators from it.

### 🟡 Q22. What is a streaming API?
The server sends the response in chunks as data becomes ready (chunked HTTP, SSE, NDJSON, gRPC streams), e.g. big exports or AI token streams. The client reads it progressively with `response.body.getReader()`.
