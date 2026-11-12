# 03 · Browser Networking and Frontend Communication

> **Subject:** Computer Networks (for full stack developers)
> **Status:** ✅ done
> **Last reviewed:** 2026-11-13

[← Back to Computer Networks](README.md) · [Questions for this topic →](questions/03-browser-networking-qa.md)

## In one line

The browser is not just a "fetch caller" — it decides **what's allowed** (Same-Origin Policy, CORS), **what's stored** (cookies, storage), and **what's reused** (cache, service worker). Most frontend network bugs come from these rules.

---

## 1. Browser networking lifecycle

> 🎤 **Say it like this:** "When JS calls fetch, the browser first checks the service worker and HTTP cache. If it must go to the network, it applies security rules like CORS, attaches cookies if allowed, resolves DNS, reuses or opens a connection, sends the request, and then checks the response against CORS before giving it back to JavaScript."

```
fetch() ─▶ Service Worker? ─▶ HTTP cache hit? ──yes──▶ return cached
                                   │ no
                                   ▼
         CORS check (preflight if needed) ─▶ attach cookies
                                   ▼
         DNS ─▶ reuse/open TCP+TLS connection ─▶ send request
                                   ▼
         response ─▶ CORS check on response ─▶ store in cache ─▶ give to JS
```

- **HTTP/1.1:** browser allows only **~6 parallel connections per host**; extra requests wait in "Queueing/Stalled". HTTP/2 removes this problem (one connection, multiplexed).

---

## 2. Fetch API

> 🎤 **Say it like this:** "fetch returns a promise that resolves with a Response object. One gotcha: it only rejects on network failure — a 404 or 500 still resolves, so you must check response.ok. Cookies are sent only for same-origin by default; for cross-origin you need credentials: 'include', and the server must allow credentials in CORS."

```js
async function getUser(id) {
  const res = await fetch(`/api/users/${id}`, {
    method: 'GET',
    headers: { 'Accept': 'application/json' },
    credentials: 'include',          // send cookies cross-origin
  });

  if (!res.ok) {                      // fetch does NOT throw on 4xx/5xx!
    throw new Error(`HTTP ${res.status}`);
  }
  return res.json();
}
```

| `credentials` value | Sends cookies |
|---|---|
| `same-origin` (default) | Only to same origin |
| `include` | Also cross-origin (needs `Access-Control-Allow-Credentials: true`) |
| `omit` | Never |

---

## 3. XMLHttpRequest (XHR)

The old way (before fetch). Callback/event based, used by older libraries.

| | XHR | fetch |
|---|---|---|
| Style | Callbacks / events | Promises |
| Upload progress | ✅ `xhr.upload.onprogress` | ❌ (no simple way) |
| Cancel | `xhr.abort()` | `AbortController` |
| Streaming body | ❌ | ✅ `res.body` (ReadableStream) |

> 💡 **Axios** uses XHR in the browser — that's why it supports upload progress easily. Axios also **throws** on 4xx/5xx, fetch doesn't.

---

## 4. Same-Origin Policy (SOP)

> 🎤 **Say it like this:** "Same-Origin Policy is a browser rule that stops JavaScript on one origin from reading responses from another origin. Origin = scheme + host + port. It protects users — so a malicious site can't read your bank data using your logged-in cookies. CORS is the controlled way for a server to relax this rule."

| URL | Same origin as `https://app.com`? |
|---|---|
| `https://app.com/users` | ✅ (path doesn't matter) |
| `http://app.com` | ❌ scheme differs |
| `https://api.app.com` | ❌ host differs (subdomain!) |
| `https://app.com:8080` | ❌ port differs |

- SOP blocks **reading** the response, not always **sending** the request (that's why CSRF exists → [06](06-auth-and-security.md)).
- `<img>`, `<script>`, `<link>`, form submit — can load cross-origin (embedding is allowed).

---

## 5. CORS (Cross-Origin Resource Sharing)

> 🎤 **Say it like this:** "CORS is a mechanism where the server tells the browser which other origins may read its responses, using Access-Control-Allow-* headers. It's enforced by the browser, not the server — Postman or curl never hit CORS errors. So the fix for a CORS error is always on the server, by sending the right headers."

### CORS headers

| Header | Sent by | Meaning |
|---|---|---|
| `Origin` | Browser | Where the request comes from |
| `Access-Control-Allow-Origin` | Server | Allowed origin (`https://app.com` or `*`) |
| `Access-Control-Allow-Methods` | Server | Allowed methods (preflight) |
| `Access-Control-Allow-Headers` | Server | Allowed custom headers (e.g. `Authorization`) |
| `Access-Control-Allow-Credentials` | Server | `true` → cookies allowed |
| `Access-Control-Max-Age` | Server | Cache the preflight result (seconds) |
| `Access-Control-Expose-Headers` | Server | Which response headers JS can read |

- ⚠️ With `credentials: 'include'`, you **cannot** use `Allow-Origin: *` — must be the exact origin.

```js
// Express
import cors from 'cors';
app.use(cors({ origin: ['https://app.com'], credentials: true }));
```

---

## 6. Preflight OPTIONS request

> 🎤 **Say it like this:** "For 'non-simple' requests — like PUT/DELETE, JSON content type, or custom headers like Authorization — the browser first sends an OPTIONS request asking the server 'is this allowed?'. Only if the server responds with the right CORS headers does the real request go. Simple requests like a plain GET or form POST skip the preflight."

| Simple request (no preflight) | Preflighted request |
|---|---|
| Method GET, HEAD, POST | PUT, PATCH, DELETE |
| Content-Type: `text/plain`, `x-www-form-urlencoded`, `multipart/form-data` | `application/json` |
| Only basic headers | Custom headers (`Authorization`, `X-Api-Key`) |

```
Browser ── OPTIONS /api/orders  (Origin, Access-Control-Request-Method: PUT) ──▶ Server
Browser ◀── 204 + Access-Control-Allow-Origin/Methods/Headers ───────────────── Server
Browser ── PUT /api/orders (real request) ─────────────────────────────────────▶ Server
```

- Preflight adds a round trip → use `Access-Control-Max-Age` to cache it.
- Common bug: auth middleware runs before CORS and rejects OPTIONS with 401 → CORS error. CORS must run first.

---

## 7. Cookies

> 🎤 **Say it like this:** "A cookie is a small piece of data the server sets with Set-Cookie, and the browser automatically sends it back on every matching request. That's why it's used for sessions. Security comes from attributes: HttpOnly so JS can't read it, Secure for HTTPS only, and SameSite to control cross-site sending."

```
Set-Cookie: sid=abc123; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=86400; Domain=.app.com
```

| Attribute | Meaning |
|---|---|
| `HttpOnly` | JS can't read it (`document.cookie`) → XSS protection |
| `Secure` | Only sent over HTTPS |
| `SameSite` | `Strict` / `Lax` (default) / `None` (needs Secure) — CSRF protection |
| `Domain` | Which host(s) get it; `.app.com` includes subdomains |
| `Path` | URL path scope |
| `Expires` / `Max-Age` | Lifetime; none = session cookie (deleted when browser closes) |

- Size limit ~4KB per cookie; sent on **every** request → keep them small.
- Security details → [06 · Auth and security](06-auth-and-security.md)

---

## 8. LocalStorage vs SessionStorage (vs Cookies vs IndexedDB)

> 🎤 **Say it like this:** "localStorage persists until cleared, sessionStorage is per tab and gone when the tab closes. Both are ~5MB, synchronous, and readable by any JS on the page — so never store sensitive tokens there because XSS can steal them. Cookies are sent automatically to the server and can be HttpOnly. IndexedDB is for larger structured data, async."

| | Cookies | localStorage | sessionStorage | IndexedDB |
|---|---|---|---|---|
| Size | ~4KB | ~5–10MB | ~5MB | Hundreds of MB+ |
| Lifetime | Expiry set by server | Forever (until cleared) | Until tab closes | Forever |
| Scope | Domain + path | Origin | Origin + **tab** | Origin |
| Sent to server automatically | ✅ | ❌ | ❌ | ❌ |
| JS access | Yes (unless HttpOnly) | Yes | Yes | Yes |
| API | String header | Sync | Sync | Async |
| Good for | Session ID, auth | Theme, preferences | Form draft per tab | Offline data, big caches |

---

## 9. Browser caching

> 🎤 **Say it like this:** "The browser caches responses based on Cache-Control. If the cache is fresh by max-age, it uses it without any request. If it's stale, it revalidates using the ETag — sends If-None-Match, and if nothing changed the server replies 304 with no body. For static assets we use hashed filenames with a long max-age and immutable, and for HTML we use no-cache so users always get the latest version."

```
Request ─▶ in cache?
            ├─ no ───────────────────────────────▶ full request → 200
            └─ yes ─▶ fresh (age < max-age)? ─yes─▶ use cache, NO request (200 from disk/memory cache)
                          │ no
                          ▼
                 revalidate: If-None-Match: "abc"
                          ├─ unchanged ─▶ 304 Not Modified (no body) → use cache
                          └─ changed ───▶ 200 + new body
```

---

## 10. Cache-Control, ETag, Last-Modified

| Directive | Meaning |
|---|---|
| `max-age=3600` | Fresh for 1 hour, no request needed |
| `no-cache` | **Can** store, but must revalidate every time (ask server) |
| `no-store` | **Never** store (sensitive data like bank pages) |
| `private` | Only browser may cache (not CDN) — user-specific data |
| `public` | CDN/proxies can cache too |
| `immutable` | Never changes, don't even revalidate on reload |
| `s-maxage` | max-age for shared caches (CDN) |

> 💡 **no-cache ≠ no caching.** no-cache = "check before using", no-store = "don't save at all". Very common interview trap.

- **ETag:** a version ID/hash of the response (`ETag: "v5-abc"`). Browser sends `If-None-Match: "v5-abc"` → `304` if same.
- **Last-Modified:** timestamp. Browser sends `If-Modified-Since` → `304` if not changed. Less precise than ETag (1-second resolution).

### Cache busting with hashed filenames

```
index.html          → Cache-Control: no-cache           (always check)
main.3f9a2c.js      → Cache-Control: max-age=31536000, immutable
styles.81bd0e.css   → Cache-Control: max-age=31536000, immutable
```

New deploy = new hash = new URL → browser fetches the new file. Old files can stay cached forever. Vite/Webpack do this automatically.

---

## 11. Service workers (basic)

> 🎤 **Say it like this:** "A service worker is a script that runs in the background, separate from the page, and sits between the page and the network like a proxy. It can intercept fetch requests and serve from cache, which enables offline support, PWAs and push notifications. It only works on HTTPS (or localhost)."

```js
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((cached) => cached || fetch(event.request))
  );
});
```

| Strategy | How | Use for |
|---|---|---|
| **Cache-first** | Cache, else network | Static assets, fonts, images |
| **Network-first** | Network, else cache (offline) | API data, news feed |
| **Stale-while-revalidate** | Return cache now, update in background | Avatars, non-critical data |
| Network-only / Cache-only | — | Payments / pre-cached app shell |

- ⚠️ A buggy service worker can keep serving an old version of your site — always have a versioning/update strategy.
- Library: **Workbox**.

---

## 12. Request cancellation using AbortController

> 🎤 **Say it like this:** "AbortController lets you cancel a fetch. You pass its signal to fetch and call abort() when you don't need the result anymore — like when a user types a new search query or a React component unmounts. This prevents race conditions where an old slow response overwrites a newer one, and saves bandwidth."

```js
const controller = new AbortController();
fetch('/api/search?q=phone', { signal: controller.signal })
  .then(r => r.json())
  .catch(err => { if (err.name === 'AbortError') return; throw err; });

controller.abort();  // cancel

// Timeout in one line (modern browsers / Node 18+)
fetch('/api/slow', { signal: AbortSignal.timeout(5000) });
```

**React — cancel on unmount or when the query changes:**
```jsx
useEffect(() => {
  const controller = new AbortController();
  fetch(`/api/search?q=${query}`, { signal: controller.signal })
    .then(r => r.json())
    .then(setResults)
    .catch(err => { if (err.name !== 'AbortError') setError(err); });

  return () => controller.abort();   // cleanup: cancel previous request
}, [query]);
```

> 💡 Aborting in the browser does **not** stop work already happening on the server — server sees a closed connection → [04 · Handling client disconnects](04-backend-networking-nodejs.md)

---

## 13. DevTools Network tab

> 🎤 **Say it like this:** "I use the Network tab to check the request URL, method, status, headers, payload and response. The Timing tab shows where time is spent — if TTFB is high, the server is slow; if content download is high, the payload is too big; if it's stuck in queueing, there are too many parallel requests. I also use throttling to test slow networks and Copy as cURL to reproduce a request outside the browser."

**Timing phases (waterfall):**

| Phase | Meaning | If it's high… |
|---|---|---|
| **Queueing / Stalled** | Waiting to be sent | Too many requests to same host (HTTP/1.1 limit) or lower priority |
| **DNS Lookup** | Resolve domain | Slow DNS; use `dns-prefetch` / `preconnect` |
| **Initial connection** | TCP handshake | Far server, no keep-alive |
| **SSL** | TLS handshake | Same as above |
| **Request sent** | Uploading the request | Large upload |
| **Waiting (TTFB)** | Time to first byte from server | **Slow backend** — DB query, API call |
| **Content Download** | Receiving body | Big payload — compress, paginate |

**Useful features:**
- **Filter** by Fetch/XHR, JS, Img, WS; search by URL.
- **Preserve log** → keep requests across page reload/redirect (debug login redirects).
- **Disable cache** → test like a first-time visitor.
- **Throttling** → simulate Slow 3G / offline.
- **Copy as cURL / fetch** → reproduce or share the exact request.
- **Initiator** → which code triggered the request.
- **Size column**: "(memory cache)" / "(disk cache)" = served from cache.
- **Red CORS error** in Console + "(failed)" / "CORS error" status in Network.

More debugging → [09 · Troubleshooting](09-troubleshooting-and-performance.md)

---

## ⚡ Quick revision

- fetch resolves on 4xx/5xx → check `res.ok`. Cookies cross-origin need `credentials: 'include'`.
- Origin = scheme + host + port. SOP blocks reading cross-origin responses.
- CORS is enforced by the **browser**; fix it on the **server**. No `*` with credentials.
- Preflight OPTIONS for PUT/DELETE, JSON body, custom headers. Cache with `Max-Age`.
- Cookies: HttpOnly, Secure, SameSite. Auto-sent, ~4KB.
- localStorage forever, sessionStorage per tab, both readable by XSS.
- `no-cache` = revalidate, `no-store` = never store. ETag → If-None-Match → 304.
- Hashed filenames + `immutable` for assets; `no-cache` for HTML.
- Service worker = proxy in the browser; cache-first vs network-first.
- AbortController cancels fetch; use in React cleanup.
- Network tab: high TTFB = slow server; high download = big payload; queueing = too many requests.
