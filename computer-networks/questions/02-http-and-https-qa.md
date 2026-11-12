# Q&A · HTTP and HTTPS

[← Back to Computer Networks](../README.md) · [Read the notes →](../02-http-and-https.md)

> Short answers you can say out loud. 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

### 🟢 Q1. What is HTTP?
HyperText Transfer Protocol — an application-layer, request-response, stateless protocol. Client sends method + URL + headers + body, server returns status + headers + body.

### 🟢 Q2. Difference between GET and POST?
GET reads data, parameters go in the URL, it's safe, idempotent and cacheable. POST sends data in the body to create or trigger something, not idempotent, not cached by default.

### 🟡 Q3. What are safe and idempotent methods?
Safe = doesn't change server state (GET, HEAD, OPTIONS). Idempotent = repeating it gives the same final state (GET, PUT, DELETE + safe ones). POST and usually PATCH are not idempotent — matters when retrying.

### 🟢 Q4. PUT vs PATCH?
PUT replaces the whole resource — missing fields get removed/reset. PATCH updates only the fields you send.

### 🟢 Q5. 401 vs 403?
401 = not authenticated (no/invalid token — "who are you?"). 403 = authenticated but not allowed ("I know you, but no").

### 🟡 Q6. 301 vs 302 vs 307 vs 308?
301/308 are permanent, 302/307 temporary. 307 and 308 guarantee the method and body are kept; with 301/302 browsers may change a POST into GET. Browsers cache 301 aggressively.

### 🟡 Q7. 400 vs 422?
400 = request is malformed (bad JSON, wrong types). 422 = format is fine but data fails validation (email already used, age negative). Many APIs just use 400 for both — be consistent.

### 🟡 Q8. 502 vs 503 vs 504?
All usually come from a proxy/load balancer. 502 = upstream gave an invalid response or closed the connection (app crashed). 503 = service unavailable / overloaded / no healthy targets. 504 = upstream took too long (slow query, timeout).

### 🟢 Q9. What is a 304 response?
"Not Modified" — the browser sent `If-None-Match` (ETag) or `If-Modified-Since`, and the resource hasn't changed, so the server sends no body and the browser uses its cache.

### 🟢 Q10. What do Content-Type and Content-Length do?
Content-Type tells how to parse the body (`application/json`, `multipart/form-data`). Content-Length is the body size in bytes so the receiver knows where it ends. Streaming responses use `Transfer-Encoding: chunked` instead.

### 🟡 Q11. Why is HTTP stateless and how do we maintain login state?
Each request is independent, so any server can handle it — easy horizontal scaling. We add state by sending a session cookie or a token (JWT) on every request.

### 🟡 Q12. HTTP/1.1 vs HTTP/2?
HTTP/2 is binary, multiplexes many requests on one TCP connection, and compresses headers. HTTP/1.1 is text, one request at a time per connection, so browsers open ~6 connections per host.

### 🔴 Q13. What is head-of-line blocking and how does HTTP/3 fix it?
In HTTP/1.1 a slow response blocks the next one on that connection. HTTP/2 fixed it at HTTP level, but on TCP one lost packet still pauses all streams. HTTP/3 runs on QUIC (UDP) where streams are independent, so loss in one stream doesn't block others.

### 🟢 Q14. HTTP vs HTTPS?
HTTPS = HTTP inside TLS on port 443. It gives encryption, integrity and server authentication via certificates. HTTP is plain text — anyone on the network can read or modify it.

### 🟡 Q15. Explain the TLS handshake.
Client sends ClientHello (ciphers + key share). Server replies with ServerHello, key share and certificate. Both derive the same session key via Diffie-Hellman, client verifies the certificate chain, then encrypted data flows. TLS 1.3 = 1 round trip.

### 🟡 Q16. Does TLS use symmetric or asymmetric encryption?
Both. Asymmetric during the handshake to verify identity and agree on a key; symmetric (AES) for the actual data because it's much faster.

### 🟡 Q17. How does the browser trust a certificate?
The cert is signed by an intermediate CA, which is signed by a root CA that's pre-installed in the OS/browser. Browser also checks domain match, expiry and revocation.

### 🔴 Q18. HTTPS works in Chrome but your Node service / curl says "unable to verify the first certificate". Why?
The server isn't sending the intermediate certificate. Browsers often fill it in themselves, Node/curl don't. Fix: configure the full chain (`fullchain.pem`) on the server.

### 🟢 Q19. What is Keep-Alive?
Reusing one TCP connection for multiple HTTP requests to skip repeated TCP + TLS handshakes. Default in HTTP/1.1.

### 🔴 Q20. Random 502s behind AWS ALB with a Node app, no errors in app logs. What's a likely cause?
Node's keep-alive timeout (5s default) is shorter than the ALB idle timeout (60s). Node closes an idle connection just as ALB reuses it → 502. Set `server.keepAliveTimeout` (and `headersTimeout`) higher than the LB timeout.

### 🟡 Q21. What is content negotiation?
Client tells what it accepts (`Accept`, `Accept-Encoding`, `Accept-Language`) and the server chooses the format/compression/language and reports it via `Content-Type`, `Content-Encoding`.

### 🟢 Q22. gzip vs Brotli?
Both compress text responses. Brotli (`br`) gives smaller output, especially for static JS/CSS; gzip is faster and universally supported. Don't compress already-compressed files like images.
