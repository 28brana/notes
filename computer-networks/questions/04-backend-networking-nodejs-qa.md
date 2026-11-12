# Q&A · Backend Networking with Node.js

[← Back to Computer Networks](../README.md) · [Read the notes →](../04-backend-networking-nodejs.md)

> Short answers you can say out loud. 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

### 🟢 Q1. TCP vs UDP?
TCP is connection-based, reliable and ordered — it retransmits lost packets. UDP has no connection and no guarantee, but is faster with less overhead. HTTP, databases and WebSockets use TCP; DNS, video calls, gaming and HTTP/3 (QUIC) use UDP.

### 🟢 Q2. Explain the TCP 3-way handshake.
Client sends SYN, server replies SYN-ACK, client sends ACK. Now both agree on sequence numbers and data can flow. It costs one round trip, which is why we reuse connections.

### 🟡 Q3. How is a TCP connection closed?
4-way close: one side sends FIN, the other ACKs, then sends its own FIN, and the first side ACKs. The closer then sits in TIME_WAIT for a while.

### 🟢 Q4. What is a socket?
One endpoint of a connection — IP + port. A server has a listening socket, and every accepted client gets its own socket. A connection is unique by client IP, client port, server IP and server port.

### 🟢 Q5. What does EADDRINUSE mean?
Another process is already bound to that port. Find it with `lsof -i :3000` and stop it, or use another port.

### 🟡 Q6. How can single-threaded Node handle thousands of connections?
JS runs on one thread, but I/O is non-blocking. While a request waits for DB or network, Node handles other requests; the OS notifies it when data is ready (event loop + epoll/kqueue). It breaks only if CPU-heavy work blocks the loop.

### 🟡 Q7. What blocks the event loop and how do you fix it?
Sync file/crypto calls, parsing huge JSON, heavy loops or bad regex. Fix: async APIs, worker threads, push work to a queue, and scale with cluster / multiple containers.

### 🟢 Q8. Explain the Express request lifecycle.
Request → middleware in registration order (logger, CORS, body parser, auth) → matching route handler → response. If something throws or calls `next(err)`, it jumps to the error middleware.

### 🟡 Q9. How is error middleware different?
It has 4 parameters `(err, req, res, next)` and must be registered after all routes. In Express 4, async errors must be passed with `next(err)` or a wrapper; Express 5 catches rejected promises.

### 🟡 Q10. Why might a request hang forever in Express?
A middleware didn't call `next()` and didn't send a response, or an awaited downstream call has no timeout.

### 🟡 Q11. What is keep-alive and why use it?
It keeps the TCP (and TLS) connection open after a response so the next request skips the handshakes. Lower latency and less CPU. For outgoing calls use an `http.Agent({ keepAlive: true })` or undici/fetch, which pool connections.

### 🔴 Q12. Random 502s behind AWS ALB, but the app looks healthy. Why?
Classic keep-alive mismatch: ALB idle timeout is 60s, Node `keepAliveTimeout` is 5s. Node closes the idle connection while ALB reuses it → 502. Set `server.keepAliveTimeout = 65000` and `headersTimeout = 66000` (greater than the LB timeout).

### 🟡 Q13. What timeouts should a backend set?
Connect timeout, read/request timeout on every outbound call (`fetch(url, { signal: AbortSignal.timeout(3000) })`), and server-side timeouts for slow clients. Inner timeouts should be shorter than outer ones (LB > API > downstream).

### 🟢 Q14. Why limit request body size?
To stop huge payloads from eating memory or blocking the event loop while parsing. `express.json({ limit: '1mb' })` returns 413. Also set `client_max_body_size` in Nginx.

### 🟡 Q15. How do you handle a client disconnecting mid-request?
Listen to `req.on('close')` and, if the response isn't finished, abort downstream work with an AbortController. Check `res.destroyed` before writing. Nginx logs these as 499.

### 🟡 Q16. Why stream responses? What is backpressure?
Streaming sends data in chunks, so memory stays low and the client gets data early. Backpressure is when the consumer is slower than the producer — `write()` returns false and you wait for `drain`. `stream.pipeline` handles it and errors automatically.

### 🟡 Q17. `pipe()` vs `pipeline()`?
`pipeline` forwards errors and destroys all streams on failure; `.pipe()` doesn't, which can leak file handles or memory. Prefer `pipeline`.

### 🔴 Q18. How do you do a graceful shutdown in Node?
On SIGTERM: fail health checks, call `server.close()` so no new connections are accepted, let in-flight requests finish, close DB/Redis, then exit. Add a force-exit timer (~10s) as a safety net.

### 🟡 Q19. Why is `req.ip` wrong behind a load balancer?
It shows the proxy's IP. The real client IP is in `X-Forwarded-For`. Set `app.set('trust proxy', 1)` so Express reads it — only trust known proxies because clients can fake that header.

### 🔴 Q20. Your Node service makes many calls to another API and you see high latency and `EMFILE`/port exhaustion. What's wrong?
It's probably opening a new connection per request (no keep-alive), leaving many sockets in TIME_WAIT. Use a keep-alive agent with `maxSockets`, add timeouts, and make sure responses are fully consumed so sockets return to the pool.

### 🟢 Q21. Why bind to `0.0.0.0` in Docker?
`127.0.0.1` inside a container only means the container itself, so port mapping from the host can't reach it. `0.0.0.0` listens on all interfaces.
