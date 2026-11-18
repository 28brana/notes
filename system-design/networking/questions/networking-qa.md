# Q&A · Networking

[← Back to Networking](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 01 · OSI Model

[Read the notes →](../01-osi-model.md)

> [!question]- 🟢 Q1. What is the OSI model and which layers matter most in system design?
> It splits networking into 7 layers, each with one job — from physical bits up to the application. In system design, L4 (TCP/UDP, IPs and ports) and L7 (HTTP, URLs, headers) matter most, because load balancers and proxies work at one of those two.

> [!question]- 🟡 Q2. What's the difference between an L4 and an L7 load balancer?
> An L4 balancer only sees IP and port, so it just forwards TCP/UDP connections — very fast and cheap. An L7 balancer reads the HTTP request, so it can route by path, host or header, terminate TLS, cache and rate limit, at a bit more CPU cost.

> [!question]- 🔴 Q3. You need to send /api to one service and /images to another. Can an L4 load balancer do it?
> No. The path is inside the HTTP request, which an L4 balancer never reads. You need an L7 balancer or reverse proxy, like Nginx, Envoy or AWS ALB.

---

## 02 · IP Address

[Read the notes →](../02-ip-address.md)

> [!question]- 🟢 Q4. What's the difference between a public and a private IP?
> A public IP is reachable from the internet; a private IP (10.x, 172.16–31.x, 192.168.x) only works inside your network. In a good design only the edge — load balancer or CDN — is public, and app servers and databases stay private.

> [!question]- 🟡 Q5. What is NAT and why do private servers need it?
> NAT lets many private machines share one public IP for outgoing traffic. Our private servers can call external APIs like a payment provider, but nobody outside can start a connection to them.

> [!question]- 🔴 Q6. You rate limit by IP and a whole office of users gets blocked. Why?
> Everyone behind the office NAT, or a mobile carrier's NAT, shares one public IP, so they look like one heavy user. Rate limit by user ID or API key instead, and use IP only as a loose backup.

---

## 03 · TCP vs UDP

[Read the notes →](../03-tcp-vs-udp.md)

> [!question]- 🟢 Q7. What's the difference between TCP and UDP?
> TCP opens a connection with a handshake and guarantees every byte arrives in order, resending lost packets. UDP just sends packets with no handshake and no guarantees — faster, but the app must handle loss.

> [!question]- 🟡 Q8. Which would you use for a video call, a payment API and DNS — and why?
> Video call: UDP, because a late frame is useless and resending only adds lag. Payment API: TCP, since every byte must arrive correctly. DNS: UDP for small, fast queries, falling back to TCP for big answers.

> [!question]- 🟡 Q9. What is head-of-line blocking and how does HTTP/3 fix it?
> In TCP, if one packet is lost, every packet behind it waits until it's resent, so one loss stalls everything. HTTP/3 runs on QUIC over UDP, which handles loss per stream, so a lost packet only delays its own stream.

> [!question]- 🔴 Q10. Your service opens a new database connection per request and latency is high. What's wrong?
> Each new connection pays the TCP handshake and often TLS and auth too — several round trips before any query runs — plus memory on the DB. Use a connection pool so connections are opened once and reused.

---

## 04 · HTTP & HTTPS

[Read the notes →](../04-http-https.md)

> [!question]- 🟢 Q11. What does it mean that HTTP is stateless, and why does it help scaling?
> The server keeps no memory between requests — each request carries what it needs, like an auth token. So any server can handle any request, and we can add servers behind a load balancer freely.

> [!question]- 🟡 Q12. What changed from HTTP/1.1 to HTTP/2 to HTTP/3?
> HTTP/1.1 handles one request at a time per connection, so browsers open several connections. HTTP/2 multiplexes many requests on one TCP connection with compressed headers. HTTP/3 moves to QUIC over UDP, removing TCP head-of-line blocking and making setup faster.

> [!question]- 🟡 Q13. Which failed requests should a client retry?
> Retry timeouts, 5xx errors and 429 (with backoff, respecting Retry-After). Don't retry 4xx like 400 or 404 — they'll fail again. And only retry non-idempotent calls like POST if you use an idempotency key.

> [!question]- 🔴 Q14. Where would you terminate TLS in a large system, and what's the trade-off?
> Usually at the load balancer or reverse proxy, so certificates live in one place and servers skip the crypto work. The trade-off is that traffic inside the network is plain unless you re-encrypt it; for sensitive or zero-trust setups, use mTLS between services.

---

## 05 · DNS

[Read the notes →](../05-dns.md)

> [!question]- 🟢 Q15. What happens during a DNS lookup?
> The browser asks a recursive resolver. If it's not cached, the resolver asks a root server, then the TLD server like .com, then the domain's authoritative server, which returns the IP. The answer is cached along the way for its TTL.

> [!question]- 🟡 Q16. What is TTL and how do you choose it?
> TTL is how long a DNS answer can be cached. Long TTL means fewer lookups but slow changes; short TTL like 60 seconds means fast failover but more lookups. Lower it a day before any planned migration.

> [!question]- 🟡 Q17. How can DNS help with global load balancing?
> GeoDNS or latency-based DNS returns the IP of the closest healthy region, so Indian users go to Mumbai and US users to Virginia. Health checks stop returning IPs of dead regions.

> [!question]- 🔴 Q18. You switched DNS to the backup region 10 minutes ago, but many users still hit the dead one. Why?
> Their resolvers, OS or apps cached the old answer and won't ask again until the TTL expires — some even ignore TTL. That's why DNS failover is slow; keep TTLs short and put a load balancer or anycast in front that can fail over in seconds.

---

## 06 · Proxy vs Reverse Proxy

[Read the notes →](../06-proxy-vs-reverse-proxy.md)

> [!question]- 🟢 Q19. What's the difference between a forward proxy and a reverse proxy?
> A forward proxy acts for clients and hides them — like a corporate proxy filtering outgoing traffic. A reverse proxy acts for servers and hides them — users hit it and it forwards to the right backend.

> [!question]- 🟡 Q20. What jobs does a reverse proxy do in a typical system?
> Load balancing, TLS termination, caching, compression, routing by path, rate limiting, and hiding server IPs. It's one front door that handles all the edge work so app servers stay simple.

> [!question]- 🟡 Q21. How is a reverse proxy different from a load balancer and an API gateway?
> A load balancer is a reverse proxy focused on spreading load. An API gateway is a reverse proxy for APIs that adds auth, API keys, rate limits and request shaping. In practice the same tool, like Nginx or Envoy, can play all three roles.

> [!question]- 🔴 Q22. After adding a reverse proxy, all your logs show the same client IP. What happened and how do you fix it?
> The servers now see the proxy's IP, not the user's. Configure the proxy to add X-Forwarded-For and read the real IP from it — but only trust that header when it comes from your own proxies, since clients can fake it.

---

## 07 · Checksums

[Read the notes →](../07-checksums.md)

> [!question]- 🟢 Q23. What is a checksum and where is it used?
> A small fingerprint of data sent or stored with it; the receiver recomputes it and a mismatch means corruption. It's used in TCP/IP headers, file uploads to object storage, disk blocks in databases, and Kafka records.

> [!question]- 🟡 Q24. Does a checksum protect data from attackers?
> No. Simple checksums like CRC only catch accidental corruption — an attacker can change the data and recompute the checksum. For tamper protection use an HMAC or a digital signature.
