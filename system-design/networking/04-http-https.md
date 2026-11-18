# 04 · HTTP & HTTPS

> [!info] Topic info
> **Subject:** System Design · **Chapter:** Networking · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Networking](README.md) · [Interview questions →](questions/networking-qa.md)

> [!abstract] In one line
> **HTTP** is the **stateless request-response** protocol of the web; **HTTPS** is HTTP inside **TLS** encryption. In design, what matters is **statelessness**, **status codes for retries**, **caching headers**, and the jump from **HTTP/1.1 → HTTP/2 → HTTP/3**.

---

## 1. The problem

Our app's feed page needs 1 HTML file, 3 API calls and 40 images. Over a slow mobile network, loading them one by one takes seconds.

And anyone on the same café Wi-Fi could read users' passwords if traffic isn't encrypted. HTTP versions fix the first problem, HTTPS fixes the second.

---

## 2. How HTTP works

The client sends a **request** (method + URL + headers + optional body). The server sends a **response** (status code + headers + body).

```mermaid
flowchart LR
    C[👥 Client] -->|GET /feed + headers| S[🖥️ Server]
    S -->|200 OK + JSON body| C
```

**Stateless** = the server remembers nothing between requests. Every request carries what it needs (like an auth token). This is what lets us add servers behind a load balancer freely — see [Scalability](../core-concepts/01-scalability.md).

### What designers use from HTTP

| Thing | Why it matters |
|---|---|
| **Methods** | `GET`, `PUT`, `DELETE` are **idempotent** — safe to retry. `POST` is not, unless you add an [idempotency key](../api-fundamentals/03-idempotency.md). |
| **Status codes** | `2xx` ok · `4xx` client's fault, **don't retry** · `5xx` server's fault, **retry with backoff** · `429` slow down |
| **Cache headers** | `Cache-Control: max-age=3600`, `ETag` → browsers and CDNs skip the server |
| **Keep-alive** | Reuse one TCP connection for many requests |

Full details: [HTTP and HTTPS](../../computer-networks/02-http-and-https.md).

---

## 3. HTTP/1.1 vs HTTP/2 vs HTTP/3

| | **HTTP/1.1** | **HTTP/2** | **HTTP/3** |
|---|---|---|---|
| **Transport** | TCP | TCP | **QUIC over UDP** |
| **Requests per connection** | One at a time | Many at once (**multiplexing**) | Many at once |
| **Headers** | Plain text, repeated | Compressed | Compressed |
| **Head-of-line blocking** | Yes, per connection | Fixed at HTTP level, still at TCP level | **Gone** — streams are independent |
| **Setup** | TCP + TLS = 2–3 round trips | Same | **1 round trip**, 0 on repeat visits |

- **HTTP/1.1:** browsers open ~6 connections per domain to work around one-at-a-time.
- **HTTP/2:** one connection carries all 40 images in parallel. Big win.
- **HTTP/3:** best on lossy mobile networks. Also survives switching Wi-Fi → 4G without reconnecting.

> [!note] Internal traffic
> Between our own services, **gRPC runs on HTTP/2** — multiplexing and binary framing make it fast. See [gRPC](../api-fundamentals/07-grpc.md).

---

## 4. HTTPS — HTTP + TLS

**TLS (Transport Layer Security)** gives three things:

- 🔒 **Encryption** — nobody in the middle can read the data.
- ✅ **Integrity** — nobody can change it in transit.
- 🪪 **Authentication** — the certificate proves you're talking to the real server.

The **TLS handshake** agrees on keys using public-key crypto, then switches to fast symmetric encryption. TLS 1.3 needs only **1 round trip**.

In real systems, TLS usually **ends at the load balancer or reverse proxy** (**TLS termination**). Internal traffic can be plain or re-encrypted (**mTLS** in zero-trust setups). See [Proxy vs reverse proxy](06-proxy-vs-reverse-proxy.md) and [SSL/TLS](../security/01-ssl-tls.md).

> [!tip] 🎤 Say it like this
> "HTTP is a stateless request-response protocol, which is what makes horizontal scaling easy. HTTP/2 multiplexes many requests on one TCP connection, and HTTP/3 moves to QUIC over UDP so one lost packet doesn't block everything. HTTPS wraps it in TLS for encryption and server identity, and we usually terminate TLS at the load balancer."

> [!failure] Interview traps
> - Retrying `4xx` errors — they'll fail again. Retry `5xx`, `429` and timeouts only.
> - Blindly retrying `POST` → duplicate orders. Use idempotency keys.
> - Saying HTTPS is "too slow". With TLS 1.3, keep-alive and session resumption the cost is tiny.
> - Thinking HTTP/2 fully fixes head-of-line blocking. TCP still has it; HTTP/3 fixes it.

---

## Summary

- HTTP is **stateless** → easy to scale out.
- Status codes tell you **what to retry**; methods tell you **what's safe to retry**.
- **HTTP/2** = multiplexing on one TCP connection. **HTTP/3** = QUIC on UDP, no HOL blocking.
- **HTTPS** = encryption + integrity + identity. Terminate TLS at the edge.

> [!success] Key takeaway
> Use HTTP/2+ everywhere, HTTPS always, cache headers for static data, and only retry what's safe.
