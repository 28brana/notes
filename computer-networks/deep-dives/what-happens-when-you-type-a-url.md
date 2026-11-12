# What Happens When You Type a URL — Full Browser → Website Flow

> **Subject:** Computer Networks  
> **Status:** ✅ done  
> **Last reviewed:** 2026-11-13

[← Back to Computer Networks](../README.md) · Short version: [01 · §7](../01-how-the-internet-works.md)

## Overview

The full journey from typing `https://www.example.com/products` and pressing Enter until the page is on screen and interactive. Interviewers ask it because one answer covers **URL parsing → caching → DNS → TCP → TLS/SSL → HTTP → servers → rendering**, and they can dig into any step.

## The whole flow at a glance

```
 You type URL + Enter
        │
 ①  Browser UI: parse input → URL or search? → HSTS check (force https)
        │
 ②  Caches: Service Worker? HTTP cache (fresh?) → if hit, skip to ⑨
        │
 ③  DNS: browser cache → OS cache / hosts file → resolver
        │      → root → TLD (.com) → authoritative NS  ⇒  IP address
        │
 ④  Getting packets out: routing table → ARP for gateway MAC → router → NAT → ISP → internet
        │
 ⑤  TCP 3-way handshake   SYN → SYN-ACK → ACK                (1 RTT)
        │
 ⑥  TLS handshake (SSL)   ClientHello(SNI, ALPN, key share)
        │                 ServerHello + Certificate + Finished
        │                 verify cert chain → session keys     (1 RTT in TLS 1.3)
        │
 ⑦  HTTP request (encrypted)   GET /products, Host, cookies, headers
        │
 ⑧  Server side: CDN edge → load balancer (TLS may end here)
        │      → reverse proxy (Nginx) → app server → cache / DB
        │
 ⑨  HTTP response: status, headers (Cache-Control, Set-Cookie…), body (HTML, gzip/br)
        │
 ⑩  Rendering: HTML → DOM, CSS → CSSOM, JS runs → render tree
        │      → layout → paint → composite  ⇒ pixels on screen
        │
 ⑪  Sub-resources (CSS, JS, images, fonts, API calls) — reuse same connection
        │
 ⑫  Page loaded (DOMContentLoaded → load) → connection kept alive for reuse
```

> 🎤 **Say it like this (30-second version):** "The browser first parses what I typed and decides it's a URL; HSTS can force it to HTTPS. It checks its caches. If it needs the network, it resolves the domain to an IP with DNS, opens a TCP connection with the three-way handshake, then does a TLS handshake where the server proves its identity with a certificate and both sides agree on a session key. Then it sends the encrypted HTTP request. The request usually passes through a CDN, a load balancer and a reverse proxy before the app server, which may hit a cache or database. The response comes back with status, headers and HTML. The browser parses the HTML into the DOM, CSS into the CSSOM, runs JavaScript, builds the render tree, then does layout, paint and compositing. Extra resources like CSS, JS and images load over the same connection, and the connection stays alive for reuse."

---

## ① Browser UI: parsing what you typed

1. **Is it a URL or a search?** The address bar (omnibox) checks: has a scheme (`https://`)? looks like a domain (`example.com`)? If not → sends it to the default search engine.
2. **Fill in what's missing:** `example.com` → `https://example.com/` (modern browsers try HTTPS first).
3. **Break the URL into parts:**

```
https://www.example.com:443/products?page=2#reviews
└─┬─┘   └──────┬──────┘ └┬┘└───┬───┘└──┬──┘└──┬──┘
scheme       host      port  path   query  fragment (never sent to server)
```

4. **Non-ASCII domains** are converted to Punycode (`münchen.de` → `xn--mnchen-3ya.de`). Special characters in path/query are percent-encoded (`space` → `%20`).
5. **HSTS check:** if the domain is in the browser's **HSTS list** (preloaded list or it earlier sent `Strict-Transport-Security`), the browser **upgrades `http://` → `https://` internally** without ever sending a plain HTTP request. That blocks SSL-stripping attacks.
6. The browser process hands the job to the **network service** (in Chrome, the browser process, network process and renderer process are separate).

> Without HSTS: `http://example.com` → server replies `301 → https://example.com` → a whole extra round trip, and the first request is unencrypted.

---

## ② Checking caches before the network

In order:

| Check | What it is | Result on hit |
|---|---|---|
| **Service Worker** | JS proxy registered by the site that can answer from its own cache (PWAs, offline) | Response with no network at all |
| **Memory cache** | Resources already in memory for this tab | Instant |
| **HTTP disk cache** | Respects `Cache-Control`, `ETag`, `Last-Modified` | **Fresh** → use directly. **Stale** → revalidate (`If-None-Match`) → `304 Not Modified` |
| **Back/forward cache** | Whole page snapshot when using Back/Forward | Instant page restore |

HTML pages are usually `Cache-Control: no-cache` (always revalidate), so for a normal navigation the browser usually still goes to the network. → Caching details in [03 · Browser Networking](../03-browser-networking.md)

---

## ③ DNS: domain → IP address

The browser needs an **IP address** for `www.example.com`.

```
Browser DNS cache
   └─▶ OS cache (+ /etc/hosts file)
         └─▶ Recursive resolver (ISP, 8.8.8.8, 1.1.1.1, or router)
               ├─▶ Root server        "ask .com servers at …"
               ├─▶ TLD server (.com)  "ask example.com's nameservers at …"
               └─▶ Authoritative NS   "www.example.com = 93.184.216.34 (TTL 300)"
   ◀── answer cached at each level for TTL seconds
```

- **Recursive resolver** does all the legwork; root → TLD → authoritative are **iterative** queries.
- Records you'll see: **A** (IPv4), **AAAA** (IPv6), **CNAME** (alias → e.g. `d123.cloudfront.net`, then resolve that too).
- DNS uses **UDP port 53** normally; **DoH** (DNS over HTTPS, 443) / **DoT** (853) encrypt the lookup.
- With a CDN, the authoritative server often returns the IP of the **nearest edge** (GeoDNS / anycast).
- Browser may also ask for an **HTTPS/SVCB record**, which can say "this site supports HTTP/3" before connecting.

Cost: 0 ms (cached) to 100+ ms (full lookup). Optimise with `<link rel="dns-prefetch" href="//api.example.com">`.

---

## ④ Getting packets out of your machine (brief)

Developers rarely get asked about this in depth, but it completes the story:

1. OS checks its **routing table**: the destination IP is not on my local network → send to the **default gateway** (router).
2. **ARP** finds the router's **MAC address** from its IP (cached in the ARP table).
3. Packet is wrapped in an Ethernet/Wi-Fi **frame** → reaches the router.
4. Router does **NAT**: swaps your private IP (`192.168.1.5`) for the public IP and keeps a mapping so replies come back to you.
5. ISP → internet backbone → many routers hop by hop (**BGP** decides routes between networks) → destination data center.

→ Layers: [OSI & TCP/IP models](osi-and-tcp-ip-models.md) · IPs/NAT: [IP addressing](ip-addressing-and-subnetting.md)

---

## ⑤ TCP 3-way handshake

TCP gives a **reliable, ordered** byte stream. The connection to `IP:443` opens with:

```
Client                              Server (port 443)
  │── SYN  (seq=x) ──────────────────▶│   "I want to talk"
  │◀── SYN-ACK (seq=y, ack=x+1) ──────│   "OK, me too"
  │── ACK  (ack=y+1) ────────────────▶│   "Connected"
```

- Costs **1 round trip (RTT)** before any data.
- The client uses a random **ephemeral port** (e.g. `52344`) → connection = `clientIP:52344 ↔ serverIP:443`.
- After this, TCP handles retransmission of lost packets, ordering, flow control and congestion control (**slow start** — the first bytes come slowly on a new connection, another reason to reuse connections).

---

## ⑥ TLS handshake — the "SSL" part

> **SSL vs TLS:** SSL is the old name; SSL 2.0/3.0 are broken and disabled. Today it's **TLS 1.2 or 1.3**, but people still say "SSL certificate".

TLS gives three things: **encryption** (nobody can read), **integrity** (nobody can change it), **authentication** (the server is who it says it is).

### TLS 1.3 handshake (1 RTT)

```
Client                                                   Server
  │── ClientHello ──────────────────────────────────────────▶│
  │     • supported TLS versions & cipher suites              │
  │     • client random                                       │
  │     • key share (client's ECDHE public value)             │
  │     • SNI = "www.example.com"  (which site I want)        │
  │     • ALPN = ["h2", "http/1.1"] (which HTTP version)      │
  │                                                           │
  │◀── ServerHello ─────────────────────────────────────────│
  │     • chosen cipher (e.g. TLS_AES_128_GCM_SHA256)         │
  │     • server key share (server's ECDHE public value)      │
  │   ── from here everything is encrypted ──                 │
  │◀── EncryptedExtensions (ALPN result: "h2") ─────────────│
  │◀── Certificate (site cert + intermediate certs) ────────│
  │◀── CertificateVerify (signed with server's private key)─│
  │◀── Finished ────────────────────────────────────────────│
  │                                                           │
  │   [client verifies certificate chain + signature]         │
  │── Finished ─────────────────────────────────────────────▶│
  │══════════ Encrypted HTTP request/response ═══════════════│
```

**How the keys work (the key idea):**
1. Both sides send a **Diffie-Hellman (ECDHE) public value**.
2. Each side combines the other's public value with its own secret → both get the **same shared secret**. The secret itself **never travels over the network**.
3. From it they derive **symmetric session keys** (AES/ChaCha20) — fast encryption for all the data.
4. A new key per connection = **forward secrecy**: stealing the server's private key later can't decrypt old recorded traffic.

**What the server's private key is used for:** not for encrypting data — only for **signing** (`CertificateVerify`) to prove it owns the certificate.

### Certificate validation (what the browser checks)

```
Root CA (pre-installed in OS/browser trust store)
   └── signs Intermediate CA cert
          └── signs www.example.com cert  ← server sends this + intermediate
```

The browser checks:
1. **Chain of trust** — each cert is signed by the one above, ending in a trusted root.
2. **Hostname match** — `www.example.com` is in the cert's **SAN** list (or a wildcard `*.example.com` covers it).
3. **Validity dates** — not expired, not yet-to-start.
4. **Not revoked** — via OCSP / **OCSP stapling** (server attaches a fresh "still valid" proof) or CRLs.
5. **Certificate Transparency** — cert is logged publicly (Chrome requires it).
6. **Signature** in `CertificateVerify` matches the cert's public key → server really has the private key.

Any failure → full-page warning like `NET::ERR_CERT_DATE_INVALID` or `ERR_CERT_COMMON_NAME_INVALID`.

### TLS 1.2 vs TLS 1.3

| | TLS 1.2 | TLS 1.3 |
|---|---|---|
| Handshake | 2 RTT | **1 RTT** |
| Resumption | Session ID / tickets (1 RTT) | PSK resumption, optional **0-RTT** |
| Key exchange | RSA or (EC)DHE | Only (EC)DHE → forward secrecy always |
| Certificate | Sent in plaintext | **Encrypted** |
| Old ciphers | Many weak options | Removed (no RC4, SHA-1, CBC…) |

- **SNI** lets many HTTPS sites share one IP — the server picks the right certificate based on the name. SNI is plaintext unless **ECH** (Encrypted Client Hello) is used.
- **ALPN** agrees on HTTP/2 vs HTTP/1.1 **inside** the handshake, so there's no extra round trip.
- **0-RTT** (resumed connections) lets the client send the request with the first message — but it can be **replayed**, so only use it for safe, idempotent requests like GET.

---

## ⑦ Sending the HTTP request

Now the browser sends the actual request (encrypted inside TLS). With HTTP/2 it's binary frames, but logically:

```http
GET /products?page=2 HTTP/2
Host: www.example.com
User-Agent: Mozilla/5.0 ...
Accept: text/html,application/xhtml+xml
Accept-Encoding: gzip, br
Accept-Language: en-IN,en
Cookie: session_id=abc123
If-None-Match: "v42"            ← only if revalidating a cached copy
Sec-Fetch-Mode: navigate
```

- **Cookies** for that domain are attached automatically (respecting `Secure`, `SameSite`, `Path`).
- The `#reviews` fragment is **not** sent.

---

## ⑧ Server side: through the infrastructure

```
Browser
  └─▶ CDN edge (Cloudflare / CloudFront)   cache hit? → respond from edge
        └─▶ Load balancer (AWS ALB)          TLS often terminated here
              └─▶ Reverse proxy (Nginx)      compression, routing, rate limits
                    └─▶ App server (Node/Express)
                          ├─▶ Cache (Redis)
                          └─▶ Database (Postgres/Mongo)
```

- **TLS termination:** the CDN or load balancer decrypts HTTPS, then talks to the backend over plain HTTP or a new TLS connection inside the private network (VPC). That's why the backend reads `X-Forwarded-For` / `X-Forwarded-Proto` for the real client IP and scheme.
- **App server:** routing → middlewares (auth, parse body, check session) → handler → queries cache/DB → builds HTML (SSR) or returns JSON.

→ Details: [07 · Frontend → Backend → DB](../07-frontend-backend-database.md), [08 · Production](../08-production-networking.md)

---

## ⑨ The HTTP response

```http
HTTP/2 200
Content-Type: text/html; charset=utf-8
Content-Encoding: br
Cache-Control: no-cache
ETag: "v43"
Set-Cookie: session_id=abc123; HttpOnly; Secure; SameSite=Lax
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy: default-src 'self'

<!DOCTYPE html><html>...
```

- **3xx** → browser follows `Location` and repeats the flow (maybe new DNS/TCP/TLS if it's a new host).
- **304** → use the cached copy.
- **4xx/5xx** → show the error page the server sent.
- Browser decompresses (`br`/`gzip`), stores cookies, notes HSTS, and caches per `Cache-Control`.
- `Content-Type: text/html` → hand it to a **renderer process** (sandboxed, one per site) for rendering.

---

## ⑩ Rendering: bytes → pixels (Critical Rendering Path)

```
HTML bytes → tokens → DOM tree ──┐
                                 ├─▶ Render tree ─▶ Layout ─▶ Paint ─▶ Composite ─▶ 🖥️
CSS bytes  → CSSOM tree ─────────┘     (visible     (size &    (pixels   (GPU layers
                                         nodes only)  position)  per layer) combined)
         JavaScript can change both DOM and CSSOM ↑
```

1. **Parse HTML → DOM.** Parsing is incremental — it starts before the whole HTML arrives (streaming).
2. **Preload scanner** looks ahead in the HTML and starts downloading `<link>`, `<script>`, `<img>` early.
3. **CSS → CSSOM.** CSS is **render-blocking**: nothing is painted until the CSSOM is ready (otherwise you'd see a flash of unstyled content).
4. **JavaScript:**
   - Plain `<script>` is **parser-blocking**: HTML parsing stops, JS downloads and runs.
   - `defer` → download in parallel, run **after** parsing, in order. ✅ best default.
   - `async` → download in parallel, run **as soon as ready** (order not guaranteed) — good for analytics.
   - `type="module"` is deferred by default.
5. **Render tree** = DOM + CSSOM, only visible nodes (`display:none` is skipped).
6. **Layout (reflow):** calculate exact size and position of every box.
7. **Paint:** fill in pixels — text, colours, borders, shadows, images — into layers.
8. **Composite:** GPU combines layers into the final screen. `transform`/`opacity` animations only need this step → smooth.

**Events:**
- `DOMContentLoaded` → HTML parsed, deferred scripts run (images may still be loading).
- `load` → everything including images, iframes and CSS has loaded.

**Metrics (Core Web Vitals):** **LCP** (largest content painted, aim < 2.5 s), **INP** (response to clicks, < 200 ms), **CLS** (layout shift, < 0.1). Also **TTFB** (time to first byte) and **FCP** (first contentful paint).

---

## ⑪ Sub-resources and connection reuse

- Every CSS, JS, image, font, and `fetch()` call is another HTTP request — it goes through ②–⑨ again, **but usually reuses the same connection**:
  - **HTTP/1.1:** keep-alive, ~6 parallel connections per host.
  - **HTTP/2:** one connection, many requests **multiplexed** as streams.
  - **HTTP/3:** same over QUIC.
- A **different domain** (e.g. `cdn.example.com`, `fonts.gstatic.com`) needs its own DNS + TCP + TLS → use `<link rel="preconnect">` for important ones.
- Cross-origin `fetch()` calls may need a **CORS preflight** (`OPTIONS`) first → [03 · Browser Networking](../03-browser-networking.md)

---

## ⑫ After the page loads

- The connection is **kept alive** for later requests, then closed after an idle timeout.
- TLS **session tickets** are saved → the next visit can resume quicker (or use 0-RTT).
- DNS answer stays cached for its **TTL**; HSTS policy stored for `max-age`.

---

## Variant: HTTP/3 over QUIC

If the server advertised HTTP/3 (via `Alt-Svc: h3=":443"` header or a DNS HTTPS record):

```
TCP + TLS 1.3:  SYN/SYN-ACK/ACK (1 RTT)  +  TLS (1 RTT)  = 2 RTT before request
QUIC (UDP):     QUIC + TLS 1.3 combined                   = 1 RTT (0-RTT on resume)
```

- Runs on **UDP 443**; TLS 1.3 is built in (no unencrypted QUIC).
- No **head-of-line blocking** between streams — one lost packet doesn't stall the others (it does in HTTP/2 over TCP).
- **Connection migration:** switching Wi-Fi → mobile data keeps the connection (uses a connection ID, not IP+port).

---

## Where the time goes (typical first visit)

| Step | Round trips | Typical cost | How to reduce |
|---|---|---|---|
| Redirect `http`→`https` | 1+ | 50–300 ms | HSTS preload, link to https directly |
| DNS | 0–several | 0–100+ ms | Long TTL, `dns-prefetch`, fast DNS provider |
| TCP | 1 | 1 RTT | CDN edge near user, keep-alive, HTTP/3 |
| TLS | 1 (1.3) / 2 (1.2) | 1–2 RTT | TLS 1.3, session resumption, OCSP stapling |
| Server (TTFB) | — | 10 ms – seconds | Caching, DB indexes, CDN for HTML |
| Download | — | depends on size | Brotli/gzip, smaller bundles |
| Render | — | depends | `defer` JS, inline critical CSS, optimise images |

Check it in **DevTools → Network → Timing** for a request: *Queueing, Stalled, DNS Lookup, Initial connection, SSL, Request sent, Waiting for server response (TTFB), Content Download.*

## See each step yourself

```bash
dig www.example.com +trace            # DNS: root → TLD → authoritative
nslookup www.example.com              # quick DNS lookup
curl -v https://www.example.com       # TCP connect, TLS handshake, cert, request/response headers
openssl s_client -connect www.example.com:443 -servername www.example.com
                                      # TLS details + full certificate chain
curl -w "dns:%{time_namelookup} tcp:%{time_connect} tls:%{time_appconnect} ttfb:%{time_starttransfer} total:%{time_total}\n" \
     -o /dev/null -s https://www.example.com   # time for each step
traceroute www.example.com            # routers hop by hop
```

Chrome: `chrome://net-internals/#dns` (DNS cache), `chrome://net-internals/#hsts` (HSTS list).

---

## Trade-offs / pitfalls

| Pitfall | What happens | Fix |
|---|---|---|
| Missing intermediate cert | Works in Chrome (it fetches/caches it), fails in curl, Node, Android | Serve the **full chain** (`fullchain.pem`) |
| Certificate expired | Hard browser error for every user | Auto-renew (Let's Encrypt / ACM) + expiry alerts |
| No HSTS | First request over plain HTTP → SSL-stripping possible | `Strict-Transport-Security` header (+ preload) |
| Mixed content | HTTPS page loading `http://` scripts → blocked | Everything over HTTPS |
| Render-blocking JS in `<head>` | Blank page while scripts download | `defer` / `async` |
| Too many third-party domains | Extra DNS+TCP+TLS per domain | Self-host, `preconnect` the critical ones |
| Low DNS TTL always | More lookups | Low TTL only around migrations |

## Quick revision

- **Order:** parse URL → HSTS → caches → **DNS** → **TCP** (1 RTT) → **TLS** (1 RTT in 1.3) → **HTTP request** → CDN/LB/proxy/app/DB → **response** → **DOM + CSSOM → render tree → layout → paint → composite**.
- HSTS = browser upgrades to HTTPS itself, no plain-HTTP request.
- DNS: browser → OS/hosts → resolver → root → TLD → authoritative; cached by TTL.
- TLS: ECDHE key share → shared secret never sent → symmetric session keys. Private key only **signs**.
- Cert check: chain to trusted root, hostname in SAN, not expired, not revoked.
- SNI = which site (cert) I want; ALPN = which HTTP version.
- TLS is often terminated at CDN/load balancer → `X-Forwarded-*` headers.
- CSS is render-blocking; plain `<script>` is parser-blocking → use `defer`.
- HTTP/3 = QUIC over UDP, TCP+TLS merged into 1 RTT, no TCP head-of-line blocking.

## Self-test questions

1. What does HSTS do in this flow, and what attack does it prevent?
   <details><summary>Answer</summary>

   It makes the browser switch `http://` to `https://` itself, before sending anything, for domains that sent the HSTS header earlier or are on the preload list. This blocks SSL stripping, where an attacker on the network intercepts the first plain-HTTP request and keeps the user on HTTP.

   </details>

2. During the TLS handshake, is the session key sent over the network? How do both sides get it?
   <details><summary>Answer</summary>

   No. Both sides exchange (EC)DHE public values. Each combines the other's public value with its own private value to compute the same shared secret, then derives symmetric session keys from it. Because the keys are new for each connection, this also gives forward secrecy.

   </details>

3. What is the server's certificate private key actually used for in TLS 1.3?
   <details><summary>Answer</summary>

   Only for signing the handshake (`CertificateVerify`), to prove the server owns the certificate. It is not used to encrypt the data or the session key.

   </details>

4. What is SNI and why is it needed?
   <details><summary>Answer</summary>

   Server Name Indication is the hostname sent in the ClientHello. Many HTTPS sites can share one IP, so the server needs the name to choose which certificate to send. It is plaintext unless Encrypted Client Hello (ECH) is used.

   </details>

5. How many round trips before the first HTTP byte on a new HTTPS connection (TLS 1.2 vs 1.3 vs HTTP/3)?
   <details><summary>Answer</summary>

   Not counting DNS: TCP (1) + TLS 1.2 (2) = **3 RTT**. TCP (1) + TLS 1.3 (1) = **2 RTT**. QUIC/HTTP/3 = **1 RTT**, or **0 RTT** when resuming.

   </details>

6. A site works in Chrome but `curl` says "unable to get local issuer certificate". Why?
   <details><summary>Answer</summary>

   The server isn't sending the intermediate certificate. Chrome can fetch or cache the missing intermediate, but curl and Node can't build the chain to a trusted root. Fix it by serving the full chain.

   </details>

7. Why is CSS render-blocking and JS parser-blocking? How do you fix slow first paint?
   <details><summary>Answer</summary>

   The browser won't paint without the CSSOM, to avoid a flash of unstyled content. A plain script can change the DOM (e.g. with `document.write`), so the parser waits for it. Fixes: `defer`/`async` scripts, inline critical CSS, preload key assets, and keep CSS small.

   </details>

8. Where is TLS usually terminated in production, and how does the app find the client's real IP?
   <details><summary>Answer</summary>

   Usually at the CDN or load balancer (sometimes Nginx). The backend gets plain HTTP or re-encrypted traffic from the proxy, so it reads `X-Forwarded-For` for the client IP and `X-Forwarded-Proto` for the original scheme. In Express, set `trust proxy`.

   </details>

## References

- [How browsers work — web.dev](https://web.dev/articles/howbrowserswork)
- [Inside look at modern web browser — Chrome Developers](https://developer.chrome.com/blog/inside-browser-part1)
- [The Illustrated TLS 1.3 Connection](https://tls13.xargs.org/)
- [What happens when… (GitHub)](https://github.com/alex/what-happens-when)
- [Critical rendering path — MDN](https://developer.mozilla.org/en-US/docs/Web/Performance/Critical_rendering_path)
