# Q&A · Browser Networking and Frontend Communication

[← Back to Computer Networks](../README.md) · [Read the notes →](../03-browser-networking.md)

> Short answers you can say out loud. 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

### 🟢 Q1. Does fetch throw an error on a 404 or 500?
No. fetch only rejects on network failure (offline, DNS, CORS block, abort). For 4xx/5xx it resolves — you must check `res.ok` or `res.status`. Axios, by contrast, throws.

### 🟢 Q2. fetch vs XMLHttpRequest vs Axios?
XHR is the old callback-based API, supports upload progress. fetch is promise-based, supports streaming and AbortController. Axios is a library (on XHR in the browser) with interceptors, auto JSON, and throws on non-2xx.

### 🟢 Q3. What is the Same-Origin Policy?
A browser rule that blocks JS on one origin from reading responses from another origin. Origin = scheme + host + port. It stops a malicious site from reading your data from other sites using your cookies.

### 🟢 Q4. What is CORS?
A way for the server to tell the browser which other origins may read its responses, via `Access-Control-Allow-*` headers. It's a controlled relaxation of SOP.

### 🟡 Q5. Why does my API work in Postman but give a CORS error in the browser?
CORS is enforced only by browsers. Postman/curl don't apply it. Fix is on the server: return the correct `Access-Control-Allow-Origin` (and methods/headers/credentials).

### 🟡 Q6. What is a preflight request and when does it happen?
An automatic `OPTIONS` request the browser sends before a "non-simple" request — methods like PUT/DELETE, `Content-Type: application/json`, or custom headers like `Authorization`. The real request is sent only if the server approves.

### 🟡 Q7. How do you reduce preflight overhead?
Set `Access-Control-Max-Age` so the browser caches the preflight result, or serve the API on the same origin (e.g. `/api` via reverse proxy) so CORS isn't needed at all.

### 🟡 Q8. Can you use `Access-Control-Allow-Origin: *` with cookies?
No. With `credentials: 'include'`, the server must return the exact origin and `Access-Control-Allow-Credentials: true`.

### 🔴 Q9. CORS was working, then you added JWT auth and now every request fails with CORS error. Why?
`Authorization` header triggers a preflight, and the auth middleware rejects the OPTIONS request (no token) with 401 without CORS headers. Put CORS middleware before auth and allow `Authorization` in `Access-Control-Allow-Headers`.

### 🟢 Q10. What are cookies and their important attributes?
Small data set by the server via `Set-Cookie` and auto-sent by the browser on matching requests. Key attributes: `HttpOnly` (no JS access), `Secure` (HTTPS only), `SameSite` (cross-site rules), `Domain`, `Path`, `Max-Age`/`Expires`.

### 🟢 Q11. localStorage vs sessionStorage vs cookies?
localStorage persists until cleared; sessionStorage lasts only for the tab; both ~5MB, JS-only, never sent to the server. Cookies are ~4KB, auto-sent with requests and can be HttpOnly.

### 🟡 Q12. Is it safe to store a JWT in localStorage?
Risky — any XSS can read it and steal the token. Safer: store it in an `HttpOnly; Secure; SameSite` cookie, and handle CSRF. If using localStorage, keep access tokens short-lived and have strong XSS protection (CSP, escaping).

### 🟡 Q13. no-cache vs no-store?
`no-cache` = may store, but must revalidate with the server before every use. `no-store` = never store at all (sensitive data).

### 🟡 Q14. How do ETags work?
Server sends `ETag: "abc"` (version/hash). Next time the browser sends `If-None-Match: "abc"`. If unchanged, server replies `304 Not Modified` with no body, saving bandwidth.

### 🔴 Q15. How do you make sure users get the new JS after a deploy but still cache aggressively?
Use content-hashed filenames (`main.3f9a.js`) with `Cache-Control: max-age=31536000, immutable`, and serve `index.html` with `no-cache`. New deploy → HTML points to new filenames → browser downloads only what changed.

### 🟢 Q16. What is a service worker?
A background script that acts as a proxy between the page and network. It can intercept requests and serve from cache — enabling offline mode, PWAs and push notifications. Needs HTTPS.

### 🟡 Q17. Cache-first vs network-first?
Cache-first: serve from cache, fall back to network — good for static assets. Network-first: try network, fall back to cache when offline — good for fresh API data. Stale-while-revalidate: serve cache now, refresh in background.

### 🟢 Q18. How do you cancel a fetch request?
Create an `AbortController`, pass `controller.signal` to fetch, call `controller.abort()`. The promise rejects with `AbortError`. `AbortSignal.timeout(ms)` gives a timeout.

### 🔴 Q19. A search box shows results for an older query sometimes. Why and how to fix?
Race condition — an older slow request finishes after the newer one and overwrites results. Abort the previous request with AbortController (React `useEffect` cleanup), and/or debounce input.

### 🟡 Q20. Why do some requests show "Stalled/Queueing" for a long time?
On HTTP/1.1 the browser allows only ~6 connections per host; extra requests wait. Fix: enable HTTP/2, reduce number of requests (bundle, lazy-load), or spread across hosts.

### 🔴 Q21. An API call takes 3 seconds. How do you find where time is spent using DevTools?
Open the request's Timing tab. High DNS/connection/SSL → network/handshake (use preconnect, keep-alive). High **Waiting (TTFB)** → backend is slow (check DB queries, logs). High **Content Download** → payload too big (compress, paginate). Long **Queueing** → too many parallel requests.

### 🟢 Q22. Useful DevTools Network features?
Preserve log (keep across redirects), Disable cache, Throttling (Slow 3G/offline), Copy as cURL (reproduce request), Initiator (which code called it), filters for Fetch/XHR and WS.
