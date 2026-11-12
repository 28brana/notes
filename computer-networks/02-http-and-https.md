# 02 · HTTP and HTTPS

> **Subject:** Computer Networks (for full stack developers)
> **Status:** ✅ done
> **Last reviewed:** 2026-11-13

[← Back to Computer Networks](README.md) · [Questions for this topic →](questions/02-http-and-https-qa.md)

## In one line

HTTP is the language browsers and servers use to talk: the client sends a **request** (method + URL + headers + body), the server sends a **response** (status + headers + body). HTTPS is the same thing wrapped inside an encrypted **TLS** tunnel.

---

## 1. HTTP request and response structure

> 🎤 **Say it like this:** "An HTTP request has a request line — method, path and version — then headers, an empty line, and an optional body. The response has a status line — version, status code and reason — then headers, an empty line, and the body. It's plain text in HTTP/1.1; HTTP/2 and 3 send the same information as binary frames."

```
POST /api/orders HTTP/1.1            ← request line (method, path, version)
Host: shop.example.com               ← headers
Content-Type: application/json
Authorization: Bearer eyJhbGci...
Content-Length: 27
                                     ← empty line
{"productId": 5, "qty": 2}           ← body
```

```
HTTP/1.1 201 Created                 ← status line
Content-Type: application/json
Location: /api/orders/981
Content-Length: 31

{"id": 981, "status": "placed"}
```

---

## 2. HTTP methods

> 🎤 **Say it like this:** "GET reads, POST creates, PUT replaces the whole resource, PATCH updates part of it, DELETE removes it. GET, PUT and DELETE are idempotent — calling them many times gives the same result as once. POST is not idempotent, that's why we use idempotency keys for payments."

| Method | Use | Body? | Safe | Idempotent |
|---|---|---|---|---|
| **GET** | Read data | No | ✅ | ✅ |
| **HEAD** | Same as GET but headers only (check size / exists) | No | ✅ | ✅ |
| **OPTIONS** | Ask what's allowed (used by CORS preflight) | No | ✅ | ✅ |
| **POST** | Create / trigger an action | Yes | ❌ | ❌ |
| **PUT** | Replace whole resource | Yes | ❌ | ✅ |
| **PATCH** | Partial update | Yes | ❌ | ❌ (usually) |
| **DELETE** | Remove | Optional | ❌ | ✅ |

- **Safe** = does not change anything on the server (read-only).
- **Idempotent** = doing it 1 time or 10 times leaves the server in the same state. Matters for **retries**: safe to retry idempotent calls automatically.

**PUT vs PATCH:**
```
Current: { name: "Bharat", city: "Delhi", age: 28 }

PUT   { name: "Bharat", city: "Pune" }  → { name: "Bharat", city: "Pune" }          (age gone!)
PATCH { city: "Pune" }                  → { name: "Bharat", city: "Pune", age: 28 }
```

---

## 3. HTTP status codes

> 🎤 **Say it like this:** "2xx is success, 3xx is redirect, 4xx means the client did something wrong, 5xx means the server failed. The ones I use most are 200, 201, 204, 301/302, 304, 400, 401, 403, 404, 409, 422, 429, 500, 502, 503 and 504."

| Code | Meaning | When to use |
|---|---|---|
| **1xx** | Informational | `101 Switching Protocols` (WebSocket upgrade) |
| **200** OK | Success | Normal GET/PUT |
| **201** Created | New resource made | After POST, with `Location` header |
| **204** No Content | Success, no body | DELETE, or update with nothing to return |
| **301** Moved Permanently | Permanent redirect | Old URL → new URL (SEO passes over, cached) |
| **302** Found | Temporary redirect | Login redirect; browser may change POST to GET |
| **304** Not Modified | Use your cache | Response to `If-None-Match` / `If-Modified-Since` |
| **307** Temporary Redirect | Like 302 but **keeps method & body** | Redirect a POST safely |
| **308** Permanent Redirect | Like 301 but **keeps method & body** | |
| **400** Bad Request | Malformed request | Invalid JSON, missing fields |
| **401** Unauthorized | **Not logged in** / bad token | "Who are you?" |
| **403** Forbidden | Logged in but **not allowed** | "I know you, but no." |
| **404** Not Found | Resource doesn't exist | Also used to hide existence of private things |
| **405** Method Not Allowed | Wrong method on route | |
| **409** Conflict | State conflict | Duplicate email, version mismatch |
| **413** Payload Too Large | Body too big | Upload over limit |
| **422** Unprocessable Entity | Valid format, invalid data | Validation errors |
| **429** Too Many Requests | Rate limited | Send `Retry-After` header |
| **500** Internal Server Error | Bug / unhandled exception | |
| **502** Bad Gateway | Proxy got a **bad/no response** from upstream | App crashed, wrong port |
| **503** Service Unavailable | Server overloaded / down for maintenance | |
| **504** Gateway Timeout | Proxy **waited too long** for upstream | Slow DB query, slow app |

> 💡 **401 vs 403** is asked in almost every interview: 401 = authentication problem, 403 = authorization problem.
> More on 502/503/504 debugging → [09 · Troubleshooting](09-troubleshooting-and-performance.md)

---

## 4. Request and response headers

Headers are key-value metadata about the request/response.

| Header | Side | Purpose |
|---|---|---|
| `Host` | Req | Which domain (one IP can host many sites) |
| `User-Agent` | Req | Which client/browser |
| `Accept` | Req | Formats client wants (`application/json`) |
| `Accept-Encoding` | Req | Compressions client supports (`gzip, br`) |
| `Authorization` | Req | Credentials (`Bearer <token>`) |
| `Cookie` / `Set-Cookie` | Req / Res | Send cookies / set cookies |
| `Content-Type` | Both | Format of the body |
| `Content-Length` | Both | Size of the body in bytes |
| `Content-Encoding` | Res | Compression used (`br`, `gzip`) |
| `Cache-Control`, `ETag` | Res | Caching rules → [03](03-browser-networking.md) |
| `Location` | Res | Where to redirect / new resource URL |
| `Origin`, `Access-Control-*` | Both | CORS → [03](03-browser-networking.md) |
| `Retry-After` | Res | When to retry (429 / 503) |
| `X-Request-Id` | Both | Trace one request across services (custom) |

---

## 5. Content-Type and Content-Length

> 🎤 **Say it like this:** "Content-Type tells the receiver how to parse the body — JSON, form data, HTML, etc. Content-Length tells how many bytes the body is, so the receiver knows where the message ends. If the size isn't known up front, like in streaming, HTTP/1.1 uses chunked transfer encoding instead."

| Content-Type | Used for |
|---|---|
| `application/json` | APIs |
| `application/x-www-form-urlencoded` | Normal HTML form (`a=1&b=2`) |
| `multipart/form-data` | File uploads → [05](05-api-communication-patterns.md) |
| `text/html; charset=utf-8` | Web pages |
| `application/octet-stream` | Raw binary / download |

- Wrong `Content-Type` → Express `req.body` is empty (`express.json()` only parses `application/json`).
- `Transfer-Encoding: chunked` → body sent in pieces, no `Content-Length` needed.

### Content negotiation & compression

- Client says what it wants: `Accept: application/json`, `Accept-Encoding: gzip, br`, `Accept-Language: en`.
- Server picks and tells: `Content-Type: application/json`, `Content-Encoding: br`.
- **Brotli (`br`)** compresses text better than **gzip**; both cut JSON/HTML/JS size by ~70–80%. Don't compress images/videos (already compressed).

---

## 6. HTTP statelessness

> 🎤 **Say it like this:** "HTTP is stateless — every request is independent and the server doesn't remember previous requests by default. To keep state like 'logged in', we send something on every request: a cookie with a session ID, or a token in the Authorization header. Statelessness is good because any server behind a load balancer can handle any request, which makes horizontal scaling easy."

- State is added on top using **cookies, sessions, JWT** → [06 · Auth](06-auth-and-security.md)
- If you store sessions in server memory, you break statelessness → need sticky sessions or a shared store like Redis.

---

## 7. HTTP/1.1 vs HTTP/2 vs HTTP/3

> 🎤 **Say it like this:** "HTTP/1.1 is text-based and handles one request at a time per connection, so browsers open ~6 connections per host and requests wait in line — head-of-line blocking. HTTP/2 is binary and multiplexes many requests over a single TCP connection, plus header compression. But if one TCP packet is lost, all streams still wait. HTTP/3 fixes that by running over QUIC on UDP, where each stream is independent, and it also has a faster handshake."

| | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---|---|---|---|
| Transport | TCP | TCP | **QUIC (over UDP)** |
| Format | Text | Binary frames | Binary frames |
| Requests per connection | 1 at a time | Many in parallel (**multiplexing**) | Many, independent streams |
| Header compression | ❌ | HPACK | QPACK |
| Head-of-line blocking | At HTTP level | Fixed at HTTP, **still at TCP** level | **Gone** |
| Handshake | TCP + TLS | TCP + TLS | Combined, 1-RTT (0-RTT on resume) |
| Connection migration (Wi-Fi → 4G) | ❌ | ❌ | ✅ |

```
HTTP/1.1:  [req1───res1][req2───res2][req3───res3]     (wait in line)
HTTP/2:    [req1][req2][req3] ─ all interleaved on one connection
```

**Practical impact:** old tricks like domain sharding and sprite sheets/bundling everything into one file are less needed with HTTP/2+.

---

## 8. HTTPS and TLS basics

> 🎤 **Say it like this:** "HTTPS is HTTP over TLS. TLS gives three things: encryption — nobody in the middle can read the data; integrity — nobody can change it without detection; and authentication — the certificate proves the server is really who it claims to be. It runs on port 443."

**Symmetric vs asymmetric in TLS:**

| | Asymmetric | Symmetric |
|---|---|---|
| Keys | Public + private key pair | One shared secret key |
| Speed | Slow | Fast |
| Used in TLS for | Handshake: verify identity, agree on key | Encrypting the actual data (AES, ChaCha20) |

👉 TLS uses **asymmetric crypto to safely agree on a symmetric key**, then uses the symmetric key for everything else (because it's fast).

---

## 9. TLS handshake and certificates

> 🎤 **Say it like this (TLS 1.3):** "The client sends a ClientHello with supported ciphers and its key share. The server replies with ServerHello, its key share, and its certificate. Both sides compute the same session key using Diffie-Hellman — the key itself never travels over the wire. The client verifies the certificate against trusted CAs, and then encrypted HTTP starts. TLS 1.3 needs only one round trip; TLS 1.2 needed two."

```
Client                                        Server
  │── ClientHello (ciphers, key share) ─────────▶│
  │◀── ServerHello (key share) + Certificate ────│
  │          + Finished (all encrypted now)      │
  │  [verify certificate chain]                  │
  │── Finished ─────────────────────────────────▶│
  │═════════ Encrypted HTTP data ═══════════════│
```

### Certificates & CA chain

- A **certificate** contains: domain name, server's **public key**, expiry date, and a **signature from a Certificate Authority (CA)**.
- **Chain of trust:**
```
Root CA (pre-installed in OS/browser)
   └── signs Intermediate CA
          └── signs your certificate (example.com)
```
- Browser checks: signed by trusted chain? domain matches? not expired? not revoked?
- Server must send the **intermediate cert** too — missing intermediate = works in Chrome, fails in curl/mobile/Node.
- **Let's Encrypt** gives free certs (90 days, auto-renew). Wildcard cert `*.example.com` covers all one-level subdomains.
- More on SSL setup → [08 · Production](08-production-networking.md)
- Where TLS fits in the full page-load flow → [What happens when you type a URL](deep-dives/what-happens-when-you-type-a-url.md) (step ⑥)

---

## 10. Persistent connections and Keep-Alive

> 🎤 **Say it like this:** "Opening a TCP + TLS connection is expensive — multiple round trips. Keep-Alive reuses the same connection for multiple requests instead of opening a new one each time. It's default in HTTP/1.1. On the backend we use the same idea with connection pooling — for outgoing HTTP calls with a keep-alive agent and for databases."

- HTTP/1.0: new connection per request. HTTP/1.1: persistent by default (`Connection: keep-alive`). `Connection: close` ends it.
- Server closes idle connections after a timeout.
- ⚠️ Classic production bug: Node's keep-alive timeout (default 5s) **shorter** than the load balancer idle timeout (ALB 60s) → Node closes the connection while the LB still reuses it → random 502s. Keep Node's `keepAliveTimeout` > LB idle timeout. → [04 · Node.js](04-backend-networking-nodejs.md)

---

## ⚡ Quick revision

- Request = method + path + headers + body. Response = status + headers + body.
- GET/PUT/DELETE idempotent, POST not. PUT replaces, PATCH updates partly.
- 2xx ok, 3xx redirect, 4xx client fault, 5xx server fault. **401 = who are you, 403 = not allowed.**
- 502 bad upstream response, 503 unavailable, 504 upstream timeout.
- Content-Type = how to parse; Content-Length = how big. Compress text with br/gzip.
- HTTP is stateless → cookies/tokens add state.
- HTTP/2 = multiplexing on one TCP; HTTP/3 = QUIC over UDP, no TCP head-of-line blocking.
- TLS = encryption + integrity + authentication. Asymmetric to agree key, symmetric for data.
- Cert chain: Root CA → Intermediate → your cert.
- Keep-Alive reuses connections; avoid repeated handshakes.
