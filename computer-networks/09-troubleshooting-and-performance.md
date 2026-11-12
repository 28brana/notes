# 09 · Networking Troubleshooting and Performance

> **Subject:** Computer Networks (for full stack developers)
> **Status:** ✅ done
> **Last reviewed:** 2026-11-13

[← Back to Computer Networks](README.md) · [Questions for this topic →](questions/09-troubleshooting-and-performance-qa.md)

## In one line

When something "doesn't work", go **layer by layer**: can I resolve the name (DNS)? Can I reach the port (TCP)? Does TLS succeed? Does the app return the right response? Most production issues are timeouts, exhausted connections or misconfigured proxies, not "the network".

---

## 1. Layer-by-layer debugging flow

> 🎤 **Say it like this:** "I debug bottom-up. First DNS — does the name resolve to the right IP? Then connectivity — is the port open and reachable? Then TLS — is the certificate valid? Then HTTP — what status and headers come back? And finally the app — logs, DB, downstream calls. That tells me quickly whether it's an infra problem or a code problem."

```
1. DNS      →  dig api.example.com            wrong / no IP?      → DNS issue
2. Network  →  ping / traceroute              host unreachable?   → routing / firewall
3. Port     →  nc -zv api.example.com 443     refused / timeout?  → service down / security group
4. TLS      →  openssl s_client -connect ...  cert error?         → expired / wrong domain
5. HTTP     →  curl -v https://...            4xx / 5xx?          → proxy / app
6. App      →  logs, APM, DB metrics          slow / errors?      → code, queries, dependencies
```

---

## 2. Connection errors (Node.js error codes)

> 🎤 **Say it like this:** "Connection refused means I reached the machine but nothing is listening on that port — it's fast and explicit. A timeout means I got no answer at all — usually a firewall or security group silently dropping packets, or the host is down. Reset means the connection was established but the other side killed it."

| Error | Meaning | Likely cause | Fix |
|---|---|---|---|
| **ECONNREFUSED** | Host reachable, nobody listening on port | App not running, wrong port, bound to `127.0.0.1`, Docker using `localhost` | Start service, check port, bind `0.0.0.0`, use service name |
| **ETIMEDOUT** | No response at all | Security group / firewall dropping, wrong IP, host down, private subnet | Open SG rule, check routes, check IP |
| **ECONNRESET** | Connection killed mid-way | Server crashed, proxy/LB idle timeout closed it, keep-alive mismatch | Align timeouts, retry idempotent calls, check server logs |
| **ENOTFOUND** | DNS lookup failed | Typo in host, DNS down, internal name used outside network | Check hostname, `dig`, check `/etc/hosts` and VPC DNS |
| **EADDRINUSE** | Port already taken (server side) | Old process still running | `lsof -i :3000` → kill it |
| **EMFILE** | Too many open files | Leaking sockets, low ulimit | Close connections, raise `ulimit -n` |

---

## 3. DNS resolution failures

- Symptoms: `ENOTFOUND`, `DNS_PROBE_FINISHED_NXDOMAIN` in browser.
- Check: `dig example.com`, `nslookup example.com`, compare against `dig @8.8.8.8 example.com`.
- Common causes: record not created / typo, recent change not propagated (TTL), domain expired, private hosted zone only visible inside the VPC, wrong `/etc/hosts` entry.
- Node.js caches nothing by default and `dns.lookup` uses a thread pool — heavy DNS usage can slow things; long-running services may keep using an old IP if a library caches it.

---

## 4. CORS errors

> 🎤 **Say it like this:** "A CORS error is the browser blocking the response, not the server failing — the request often reached the server fine. I check the Network tab for the preflight OPTIONS request, then confirm the server returns the right `Access-Control-Allow-*` headers for that exact origin. Postman works because CORS is only enforced by browsers."

**Checklist:**
1. Look at the **OPTIONS preflight** in Network tab — did it return 2xx with CORS headers?
2. `Access-Control-Allow-Origin` matches the exact origin (scheme + host + port).
3. Using cookies? → `credentials: 'include'` on client **and** `Access-Control-Allow-Credentials: true` **and** origin cannot be `*`.
4. Custom headers (`Authorization`, `X-…`) listed in `Access-Control-Allow-Headers`.
5. Method listed in `Access-Control-Allow-Methods`.
6. Is the error actually a **500/404 hiding behind CORS**? Error responses from proxies/crashes often lack CORS headers.
7. Proxy/Nginx/API Gateway stripping headers or not handling OPTIONS.

Details: [Browser networking → CORS](03-browser-networking.md)

---

## 5. 502, 503 and 504 errors

> 🎤 **Say it like this:** "All three come from a proxy or load balancer, not the browser. 502 means the proxy got an invalid response from the backend — usually the app crashed or closed the connection. 503 means no healthy backend is available — overloaded, deploying or failing health checks. 504 means the backend was too slow and the proxy gave up waiting."

| Code | Meaning | Real causes | Where to look |
|---|---|---|---|
| **502 Bad Gateway** | Proxy got a bad/no response from upstream | App crashed, wrong upstream port, app closed keep-alive connection that LB reused | App logs, process restarts, Nginx error log |
| **503 Service Unavailable** | No healthy backend / server overloaded | All targets failing health checks, deploy in progress, rate limited, maintenance | Target group health, autoscaling, `/health` endpoint |
| **504 Gateway Timeout** | Upstream too slow | Slow DB query, slow third-party API, timeout on proxy shorter than the work | Slow query log, APM traces, proxy timeouts |

**The classic keep-alive 502 bug (Node behind ALB):**
ALB idle timeout = 60s, Node `server.keepAliveTimeout` default = 5s. Node closes the idle connection, ALB reuses it at the same moment → **random 502s**.

```js
const server = app.listen(3000);
server.keepAliveTimeout = 65_000;   // > LB idle timeout (60s)
server.headersTimeout   = 66_000;   // must be > keepAliveTimeout
```

---

## 6. Slow API requests

> 🎤 **Say it like this:** "First I find where the time goes. In the Network tab I check TTFB versus content download — high TTFB means the server is slow, long download means the payload is big or bandwidth is low. On the server I use logs or APM traces to see which part is slow: DB query, external API, or CPU work."

**Read the browser timing:**

| Phase | If high → |
|---|---|
| Queued / Stalled | Too many requests to same host (HTTP/1.1 limit of ~6), main thread busy |
| DNS lookup | DNS slow; use `dns-prefetch` / `preconnect` |
| Initial connection / SSL | New connection each time — enable keep-alive, HTTP/2 |
| **Waiting (TTFB)** | **Server is slow** — look at backend |
| Content download | Payload too large — paginate, compress, trim fields |

**Backend checklist:**
- **N+1 queries** (loop of queries) → join / batch / DataLoader.
- **Missing index** → `EXPLAIN ANALYZE`.
- **No connection pooling** → new DB connection per request (see [07](07-frontend-backend-database.md)).
- Sequential external calls that could be parallel → `Promise.all`.
- Large payloads → pagination, gzip/brotli, select only needed fields.
- **Cold starts** (Lambda, scaled-to-zero containers).
- Blocking the event loop (sync crypto, big JSON parse) — see [04](04-backend-networking-nodejs.md).
- No caching for repeated reads → Redis / HTTP caching.
- Far-away region → CDN / closer deployment.

---

## 7. Latency, bandwidth, throughput

> 🎤 **Say it like this:** "Latency is how long one trip takes, bandwidth is how much data the pipe can carry at once, and throughput is how much actually gets through. For most web APIs, latency matters more than bandwidth — many small round trips add up, so we reduce round trips with keep-alive, HTTP/2, batching and CDNs."

| Term | Meaning | Analogy |
|---|---|---|
| **Latency** | Time for data to travel (ms). RTT = there and back | Length of the road |
| **Bandwidth** | Max capacity (Mbps) | Number of lanes |
| **Throughput** | Actual achieved rate | Cars that actually pass per hour |

**Latency intuition (rough):**

| Operation | Time |
|---|---|
| Memory read | ~100 ns |
| Redis call (same region) | ~0.5–1 ms |
| Simple DB query (same region) | ~1–5 ms |
| Same-region network round trip | ~0.5–2 ms |
| Mumbai ↔ Singapore | ~60 ms |
| India ↔ US East | ~200–250 ms |
| New HTTPS connection (DNS + TCP + TLS) | 2–4 round trips extra |

👉 A new HTTPS connection to the US from India can cost ~0.5–1s before the first byte. That's why keep-alive, connection reuse and CDNs matter.

---

## 8. Packet loss (conceptual)

- Some packets never arrive (bad Wi-Fi, congested links, mobile networks).
- **TCP** notices and **re-sends** them → reliable but slower; it also shrinks its sending rate (congestion control). So even 1–2% loss can make things feel very slow.
- **Head-of-line blocking:** in TCP, one lost packet blocks everything behind it — HTTP/3 (QUIC over UDP) fixes this per stream.
- **UDP** doesn't re-send → used where late data is useless (video calls, games).
- Check: `ping` shows `% packet loss`; `mtr` shows where along the path.

---

## 9. Retries and exponential backoff

> 🎤 **Say it like this:** "I retry only transient failures — timeouts, connection resets, 502/503/504, 429 — and only for idempotent operations, or with an idempotency key. I use exponential backoff with jitter so retries spread out and don't hammer an already struggling service, and I cap the number of attempts."

```js
async function withRetry(fn, { retries = 3, base = 200, max = 5000 } = {}) {
  for (let attempt = 0; ; attempt++) {
    try {
      return await fn();
    } catch (err) {
      const retryable = ['ECONNRESET', 'ETIMEDOUT'].includes(err.code)
        || [429, 502, 503, 504].includes(err.status);
      if (!retryable || attempt >= retries) throw err;

      const backoff = Math.min(max, base * 2 ** attempt);   // 200, 400, 800, ...
      const delay = Math.random() * backoff;                 // full jitter
      await new Promise(r => setTimeout(r, delay));
    }
  }
}
```

- **Don't retry:** 400, 401, 403, 404, 422 — retrying won't change the answer.
- **Don't blindly retry POST** (e.g. payments) → duplicate charges. Use idempotency keys ([05](05-api-communication-patterns.md)).
- Respect `Retry-After` header on 429/503.
- Retries at every layer multiply: 3 retries × 3 layers = 27 calls. Retry in one place.

---

## 10. Circuit breakers

> 🎤 **Say it like this:** "A circuit breaker stops calling a dependency that keeps failing. After a threshold of failures it opens and fails fast instead of waiting on timeouts, which protects my service's threads and connections. After a cooldown it goes half-open and lets a few test requests through — if they succeed it closes again."

```
          failures > threshold
 CLOSED ───────────────────────▶ OPEN  (fail fast, return fallback)
   ▲                               │
   │ test calls succeed            │ after cooldown (e.g. 30s)
   │                               ▼
   └──────────────────────── HALF-OPEN (allow few test calls)
                    test fails → back to OPEN
```

- Gives the failing service time to recover and stops cascading failures.
- Return a fallback: cached data, default value, "try later" message.
- Node library: `opossum`. Service meshes (Istio/Envoy) can do it too.

---

## 11. Timeouts everywhere

> 🎤 **Say it like this:** "Every network call must have a timeout — default is often infinite or very long. Without it, slow dependencies hold connections open, pools fill up, and the whole service hangs. I also keep a timeout budget: inner calls must time out before the outer ones."

```
Browser (30s) > CDN/ALB (60s idle) > Nginx proxy_read_timeout > API handler (10s)
                                                               > DB query (5s), external API (3s)
```

- Outer timeout should be **larger** than inner ones, else you get 504 while the work continues uselessly.
- Use `AbortController` / `AbortSignal.timeout(3000)` for `fetch`, `statement_timeout` for Postgres, client timeouts for Redis and HTTP libs.

---

## 12. Connection exhaustion

> 🎤 **Say it like this:** "Connection exhaustion is when we run out of connections — DB pool slots, open file descriptors or ephemeral ports — so new requests wait or fail. It usually comes from leaks, no pooling, or slow queries holding connections too long. The fix is pooling, timeouts, releasing connections in `finally`, and sizing the pool based on DB limits divided by number of instances."

| Resource | Symptom | Cause | Fix |
|---|---|---|---|
| **DB pool** | Requests hang, "timeout acquiring connection", "too many clients" | Leaked connections, slow queries, pool × instances > DB `max_connections` | Release in `finally`, query timeouts, PgBouncer/RDS Proxy |
| **File descriptors** | `EMFILE` | Sockets/files never closed | Close resources, raise `ulimit -n` |
| **Ephemeral ports** | `EADDRNOTAVAIL`, connect errors under load | New outbound connection per request (no keep-alive), many in `TIME_WAIT` | Use keep-alive agent / connection pooling |

---

## 13. Thundering herd

- Many clients hit the same thing **at the same moment**: cache key expires and 1,000 requests all go to the DB, or all clients reconnect right after an outage.
- Fixes: jitter on retries and TTLs, request coalescing (only one request rebuilds the cache, others wait), "stale-while-revalidate", locks, and gradual reconnect.

---

## 14. Browser Network tab debugging

- **Filter** by Fetch/XHR, search URL, tick **Preserve log** (keeps requests across redirects/reloads), **Disable cache**.
- Check **Status**, **Headers** (request + response, CORS, cache, cookies), **Payload**, **Response**, **Timing** (TTFB vs download).
- `(failed)` / `net::ERR_…` → network level; red CORS message in Console → missing headers.
- Throttle to "Slow 3G" to test bad networks.
- Right click → **Copy as cURL** → reproduce outside the browser.

---

## 15. Command cheat sheet

```bash
# HTTP
curl -v https://api.example.com/users            # full request/response + TLS info
curl -I https://example.com                      # headers only
curl -X POST -H "Content-Type: application/json" -d '{"a":1}' https://api.example.com/x
curl -o /dev/null -s -w "dns:%{time_namelookup} connect:%{time_connect} tls:%{time_appconnect} ttfb:%{time_starttransfer} total:%{time_total}\n" https://example.com

# DNS
dig example.com                    # full answer + TTL
dig +short example.com             # just IPs
dig @8.8.8.8 example.com           # ask a specific resolver
nslookup example.com

# Reachability
ping example.com                   # is host alive, latency, packet loss (ICMP may be blocked!)
traceroute example.com             # path hops (mtr = live version)
nc -zv db.internal 5432            # is the port open?

# Local ports / processes
lsof -i :3000                      # who is using port 3000
ss -tulpn   /   netstat -an        # listening ports & connections

# TLS
openssl s_client -connect example.com:443 -servername example.com   # cert chain, expiry, errors
```

---

## 16. Wireshark basics

- Captures every packet on an interface — use when logs don't explain it (TLS failures, resets, weird timeouts).
- Useful display filters:

| Filter | Shows |
|---|---|
| `http` | Plain HTTP traffic |
| `dns` | DNS queries and replies |
| `tcp.port == 443` | HTTPS traffic |
| `ip.addr == 10.0.2.15` | Traffic to/from one host |
| `tcp.flags.reset == 1` | Connection resets |
| `tcp.analysis.retransmission` | Retransmits (packet loss) |

- HTTPS content is encrypted — you see handshakes and timings, not bodies. On servers, use `tcpdump` and open the file in Wireshark.

---

## ⚡ Quick revision

- Debug in order: DNS → port → TLS → HTTP → app.
- Refused = nobody listening; Timeout = dropped (firewall/SG); Reset = killed mid-way; ENOTFOUND = DNS.
- 502 bad upstream response, 503 no healthy backend, 504 upstream too slow.
- Node behind ALB: `keepAliveTimeout` > LB idle timeout.
- CORS error = browser blocked it; check preflight + exact origin + credentials rules.
- Slow API: TTFB high → server; download high → payload.
- Latency matters more than bandwidth for APIs; reduce round trips.
- Retry only transient + idempotent, with exponential backoff + jitter.
- Circuit breaker: closed → open → half-open.
- Every call needs a timeout; inner < outer.
- Exhaustion: DB pool, file descriptors, ephemeral ports → pooling + keep-alive + release.
