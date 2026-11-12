# 04 · Backend Networking with Node.js

> **Subject:** Computer Networks (for full stack developers)
> **Status:** ✅ done
> **Last reviewed:** 2026-11-13

[← Back to Computer Networks](README.md) · [Questions for this topic →](questions/04-backend-networking-nodejs-qa.md)

## In one line

Your Node server is a process that **binds to an IP + port**, accepts **TCP connections**, reads **HTTP requests** off them and writes responses back. Most production bugs here are about **timeouts, keep-alive, limits and disconnects** — not about the code logic.

---

## 1. TCP vs UDP (practical differences)

> 🎤 **Say it like this:** "TCP is connection-based and reliable — it guarantees order and delivery, retransmits lost packets, and controls flow. UDP just fires packets with no guarantee, so it's faster and lighter. I use TCP for anything where correctness matters — HTTP, databases, Redis. UDP is used where speed beats perfection — video calls, gaming, DNS, and HTTP/3 (QUIC builds reliability on top of UDP)."

| | TCP | UDP |
|---|---|---|
| Connection | Yes (handshake first) | No |
| Delivery guarantee | ✅ Yes, retransmits | ❌ No |
| Order guaranteed | ✅ Yes | ❌ No |
| Speed / overhead | Slower, more overhead | Faster, tiny header |
| Used by | HTTP/1.1, HTTP/2, DBs, SSH, WebSocket | DNS, video/voice, gaming, QUIC (HTTP/3) |
| Analogy | Phone call | Sending postcards |

---

## 2. IP addresses and port binding

> 🎤 **Say it like this:** "When a server starts, it asks the OS to bind to an IP and port. Only one process can bind the same IP:port, otherwise you get EADDRINUSE. Binding to 127.0.0.1 means only local traffic; 0.0.0.0 means accept on every interface."

```js
app.listen(3000, '0.0.0.0', () => console.log('listening on 3000'));
```

- **`EADDRINUSE`** → port already taken (old process still running, or two apps on same port).
  Fix: `lsof -i :3000` → kill it, or use another port.
- **`EACCES`** on port < 1024 → needs root. In prod, run Node on 3000 and put Nginx/LB on 80/443.
- Inside Docker/K8s → bind `0.0.0.0`, not `localhost`.

---

## 3. TCP handshake (conceptual)

> 🎤 **Say it like this:** "Before any data, TCP does a 3-way handshake: client sends SYN, server replies SYN-ACK, client sends ACK — now both sides agree on sequence numbers and the connection is open. Closing uses a 4-way close with FIN and ACK from each side. This handshake costs one round trip, which is why we reuse connections with keep-alive."

```
Open (3-way)                 Close (4-way)
Client        Server         Client        Server
  │── SYN ──────▶│             │── FIN ──────▶│
  │◀── SYN-ACK ──│             │◀── ACK ──────│
  │── ACK ──────▶│             │◀── FIN ──────│
  │  (data flows)│             │── ACK ──────▶│
```

- Handshake = **1 RTT** (round trip). With TLS on top, add 1 more RTT (TLS 1.3). See [02 · HTTP and HTTPS](02-http-and-https.md).
- `TIME_WAIT`: after closing, the side that closed first keeps the socket for a while. Thousands of short-lived connections → port exhaustion. Another reason for keep-alive.

---

## 4. Sockets (conceptual + basic implementation)

> 🎤 **Say it like this:** "A socket is one end of a connection — identified by IP + port on both sides. A server has one listening socket, and every accepted client gets its own socket. In Node, `net` gives raw TCP sockets, and `http` is built on top of it."

A connection is unique by 4 values: **client IP, client port, server IP, server port**. That's how one server port 443 serves thousands of clients.

**Raw TCP server (`net`):**
```js
const net = require('net');

const server = net.createServer((socket) => {
  socket.write('hello\n');
  socket.on('data', (chunk) => socket.write(`echo: ${chunk}`));
  socket.on('end', () => console.log('client left'));
});
server.listen(4000);
// test: nc localhost 4000
```

**HTTP server (`http`) — same thing, but Node parses HTTP for you:**
```js
const http = require('http');

http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ ok: true }));
}).listen(3000);
```

---

## 5. Node.js event loop — why one thread handles many connections

> 🎤 **Say it like this:** "Node runs JavaScript on a single thread but does I/O non-blocking. When a request waits on DB or network, Node doesn't sit idle — it registers a callback and moves to the next request. The OS (epoll/kqueue) tells Node when data is ready. So one thread can handle thousands of concurrent connections, as long as I don't block it with CPU-heavy work."

```
Request A ──▶ start DB query ──(waiting...)──────────▶ callback → respond A
Request B ──▶ start HTTP call ──(waiting...)──▶ callback → respond B
Request C ──▶ respond C
         (single JS thread never waits)
```

- **Blocks the loop (bad):** `JSON.parse` of huge payloads, sync crypto, `fs.readFileSync` in a route, big loops, heavy regex.
- Fix: worker threads, offload to a queue, or split work. Scale across CPU cores with cluster / PM2 / multiple containers.

---

## 6. Express request-response lifecycle

> 🎤 **Say it like this:** "A request comes in, Express runs middleware in the order they were registered — logging, CORS, body parsing, auth — then the matching route handler sends the response. If anything calls `next(err)` or throws, Express skips to the error middleware."

```
Client
  │  HTTP request
  ▼
[Node http server]  parses request line + headers
  ▼
Middleware 1: logger / request-id
  ▼
Middleware 2: helmet, cors
  ▼
Middleware 3: express.json({ limit })   ← body parsed here
  ▼
Middleware 4: auth (req.user = ...)
  ▼
Router → route handler → service → DB
  ▼
res.json(...)  ──────────▶ response to client
        │
   error? next(err) ──▶ Error middleware (err, req, res, next) ──▶ 500 / 4xx
```

---

## 7. Middleware and request handling

- Middleware = `(req, res, next)`. It can **modify req/res**, **end the response**, or **call `next()`**.
- **Order matters**: body parser must come before routes that read `req.body`; auth before protected routes; error handler **last**.
- Forgetting `next()` or a response → request **hangs** until timeout.

```js
app.use(requestId);
app.use(cors());
app.use(express.json({ limit: '1mb' }));
app.use('/api', authMiddleware, apiRouter);

// error middleware: 4 args, registered last
app.use((err, req, res, next) => {
  console.error(err);
  res.status(err.status || 500).json({ error: 'Something went wrong' });
});
```

> 💡 Express 4 doesn't catch rejected promises in async handlers — wrap them or use Express 5 (which does).

---

## 8. Keep-alive and connection pooling

> 🎤 **Say it like this:** "Opening a TCP + TLS connection is expensive, so we reuse connections. Keep-alive keeps the connection open after a response for the next request. For outgoing calls from Node, I use an HTTP agent with keepAlive and for databases I use a connection pool, so requests borrow an already-open connection instead of creating a new one."

**Incoming (server side):**
```js
const server = app.listen(3000);
server.keepAliveTimeout = 65_000;   // keep idle connection open 65s
server.headersTimeout   = 66_000;   // must be > keepAliveTimeout
```

**🔥 The classic 502 bug:**
AWS ALB idle timeout = **60s**. Node's default `keepAliveTimeout` = **5s**. Node closes the idle connection, but the ALB thinks it's still open and sends a request on it → **random 502s**.
**Rule:** Node `keepAliveTimeout` **>** load balancer idle timeout. `headersTimeout` > `keepAliveTimeout`.

**Outgoing (calling other APIs):**
```js
const https = require('https');
const agent = new https.Agent({ keepAlive: true, maxSockets: 50 });
axios.get(url, { httpsAgent: agent });
```
(Node 18+ global `fetch` / undici already pools connections.)

**DB pooling:** see [07 · Frontend → Backend → Database](07-frontend-backend-database.md).

---

## 9. Timeouts

> 🎤 **Say it like this:** "Every network call must have a timeout, otherwise one slow dependency holds requests and connections forever and the whole service goes down. I set a connect timeout, a total request timeout, and also server-side timeouts so slow clients can't hold sockets."

| Timeout | Meaning |
|---|---|
| **Connect timeout** | Max time to establish TCP/TLS connection |
| **Read / socket timeout** | Max time waiting for data once connected |
| **Request (total) timeout** | Max time for the whole call |
| **Server `requestTimeout`** | Max time a client can take to send the full request (Node default 300s) |

```js
// outgoing call with a 3s total timeout
const res = await fetch(url, { signal: AbortSignal.timeout(3000) });

// cancel the call if the client already left
const ac = new AbortController();
req.on('close', () => ac.abort());
await fetch(url, { signal: ac.signal });
```

- Timeouts should get **shorter as you go deeper**: LB 60s > API 30s > downstream call 5s. Otherwise the outer layer gives up while inner work continues.
- Pair timeouts with retries + backoff → [09 · Troubleshooting](09-troubleshooting-and-performance.md).

---

## 10. Request size limits

> 🎤 **Say it like this:** "I always cap the request body size. Without it, someone can send a huge JSON and eat all memory or block the event loop parsing it. In Express that's `express.json({ limit: '1mb' })`, and Nginx has `client_max_body_size`. Large files go through streaming or presigned S3 URLs, not JSON."

```js
app.use(express.json({ limit: '1mb' }));            // too big → 413 Payload Too Large
app.use(express.urlencoded({ extended: true, limit: '1mb' }));
```
- Also limit at the proxy (Nginx `client_max_body_size 10m;`), and limit file sizes in multer.

---

## 11. Handling client disconnects

> 🎤 **Say it like this:** "If the user closes the tab or cancels, the socket closes but my handler may still be doing expensive work. I listen to the `close` event and abort downstream work, and I check `res.writableEnded` before writing so I don't write to a dead socket."

```js
app.get('/report', async (req, res) => {
  const ac = new AbortController();
  req.on('close', () => {
    if (!res.writableEnded) ac.abort();   // client left before we finished
  });

  const data = await buildReport({ signal: ac.signal });
  if (!res.destroyed) res.json(data);
});
```
- Common log: `ECONNRESET` / `EPIPE` → other side closed the connection. Usually safe to ignore for clients, investigate for upstreams.
- Nginx logs status **499** = client closed request before server responded.

---

## 12. Streaming responses

> 🎤 **Say it like this:** "Instead of loading a whole file or result in memory and sending it at once, I stream it in chunks. Memory stays flat and the client starts receiving early. I use `pipeline` because it handles errors and backpressure — if the client is slow, it pauses reading from the source."

```js
const { pipeline } = require('stream/promises');
const fs = require('fs');
const zlib = require('zlib');

app.get('/export', async (req, res) => {
  res.setHeader('Content-Type', 'text/csv');
  res.setHeader('Content-Encoding', 'gzip');
  await pipeline(fs.createReadStream('big.csv'), zlib.createGzip(), res);
});
```

- **Backpressure:** `res.write()` returns `false` when the buffer is full → wait for `'drain'`. `pipe`/`pipeline` do this for you.
- Prefer `pipeline` over `.pipe()` — `.pipe()` doesn't forward errors or clean up properly.
- Without `Content-Length`, HTTP/1.1 sends it with **`Transfer-Encoding: chunked`**.

---

## 13. Graceful shutdown

> 🎤 **Say it like this:** "On deploy, the container gets SIGTERM. I stop accepting new connections with `server.close()`, let in-flight requests finish, then close the DB and Redis connections and exit. A force-exit timer makes sure it doesn't hang forever. Otherwise users see 502s on every deploy."

```js
process.on('SIGTERM', () => {
  server.close(async () => {          // stop new connections, wait for active ones
    await db.end();
    await redis.quit();
    process.exit(0);
  });
  setTimeout(() => process.exit(1), 10_000).unref(); // safety net
});
```
- Also fail the health check first so the load balancer stops sending traffic.
- `server.closeIdleConnections()` (Node 18.2+) closes idle keep-alive sockets so `close()` finishes faster.

---

## 14. Behind a proxy: `trust proxy` and X-Forwarded-For

> 🎤 **Say it like this:** "Behind Nginx or a load balancer, `req.ip` is the proxy's IP, not the user's. The proxy adds `X-Forwarded-For` with the real client IP and `X-Forwarded-Proto` for https. I set Express `trust proxy` so `req.ip` and `req.secure` are correct — but only trust known proxies, because clients can fake that header."

```js
app.set('trust proxy', 1);   // trust 1 hop (the LB)
// X-Forwarded-For: <client>, <proxy1>, <proxy2>
```
Needed for: rate limiting by IP, logs, `secure` cookies, redirect to https.

---

## ⚡ Quick revision

- TCP = reliable + ordered (HTTP, DB). UDP = fast, no guarantee (DNS, video, QUIC).
- Bind `0.0.0.0` in containers; `EADDRINUSE` = port taken.
- 3-way handshake SYN → SYN-ACK → ACK; costs 1 RTT → reuse connections.
- Socket = one end of a connection; unique by (client IP, client port, server IP, server port).
- Node: single JS thread + non-blocking I/O; never block the loop.
- Express: middleware in order → route → error middleware (4 args, last).
- **Node `keepAliveTimeout` > LB idle timeout**, `headersTimeout` > `keepAliveTimeout`.
- Every outbound call needs a timeout (`AbortSignal.timeout`); use keep-alive agents/pools.
- Limit body size (`express.json({ limit })` → 413).
- Watch `req.on('close')` to stop work for gone clients.
- Stream with `pipeline` → low memory + backpressure handled.
- SIGTERM → `server.close()` → close DB → exit.
- `trust proxy` + `X-Forwarded-For` to get real client IP.
