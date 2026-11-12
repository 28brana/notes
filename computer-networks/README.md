# Computer Networks — for Full Stack Developers

[← Back to all subjects](../README.md)

Networking that every **senior full stack developer** is expected to know — how a request travels from the browser to the database and back, how to secure it, deploy it, and debug it.
Not router/switch-level network engineering.

**How to use:** read a topic → then answer its questions out loud without looking. Each note has 🎤 *"Say it like this"* blocks you can use almost word-for-word in an interview.

## Topics

| # | Topic | What you'll be able to answer | Q&A | Status |
|---|---|---|---|---|
| 01 | [How the Internet & Web Work](01-how-the-internet-works.md) | IP, ports, DNS, "what happens when you type a URL" | [Q&A](questions/01-how-the-internet-works-qa.md) | ✅ |
| 02 | [HTTP & HTTPS](02-http-and-https.md) | Methods, status codes, headers, HTTP/2/3, TLS | [Q&A](questions/02-http-and-https-qa.md) | ✅ |
| 03 | [Browser Networking](03-browser-networking.md) | fetch, CORS, cookies, storage, caching, DevTools | [Q&A](questions/03-browser-networking-qa.md) | ✅ |
| 04 | [Backend Networking with Node.js](04-backend-networking-nodejs.md) | TCP/UDP, sockets, Express lifecycle, timeouts, streams | [Q&A](questions/04-backend-networking-nodejs-qa.md) | ✅ |
| 05 | [API Communication Patterns](05-api-communication-patterns.md) | REST, GraphQL, WebSockets, SSE, webhooks, pagination | [Q&A](questions/05-api-communication-patterns-qa.md) | ✅ |
| 06 | [Authentication & Security](06-auth-and-security.md) | Sessions vs JWT, cookies, CSRF, XSS, OAuth, rate limiting | [Q&A](questions/06-auth-and-security-qa.md) | ✅ |
| 07 | [Frontend → Backend → Database](07-frontend-backend-database.md) | Pooling, transactions, Redis, queues, Kafka, microservices | [Q&A](questions/07-frontend-backend-database-qa.md) | ✅ |
| 08 | [Production & Deployment Networking](08-production-networking.md) | Nginx, load balancers, CDN, AWS VPC, subnets, ALB | [Q&A](questions/08-production-networking-qa.md) | ✅ |
| 09 | [Troubleshooting & Performance](09-troubleshooting-and-performance.md) | 502/503/504, timeouts, retries, circuit breakers, curl | [Q&A](questions/09-troubleshooting-and-performance-qa.md) | ✅ |

All questions in one place: [questions/README.md](questions/README.md)

## Optional deep dives

More theory-heavy; good to skim, rarely asked in depth for developer roles.

- [What happens when you type a URL — full flow](deep-dives/what-happens-when-you-type-a-url.md) — URL parsing, HSTS, DNS, TCP, TLS/SSL, server, rendering, client-side vs server-side (CSR/SSR/SSG) ⭐
- [OSI & TCP/IP models](deep-dives/osi-and-tcp-ip-models.md)
- [IP addressing & subnetting](deep-dives/ip-addressing-and-subnetting.md) — useful for AWS VPC/CIDR

## Top 10 questions to prepare first

1. What happens when you type a URL and press Enter? → [01](01-how-the-internet-works.md) · [full flow](deep-dives/what-happens-when-you-type-a-url.md)
2. HTTP vs HTTPS, and how does the TLS handshake work? → [02](02-http-and-https.md)
3. What is CORS and why does a preflight request happen? → [03](03-browser-networking.md)
4. Session vs JWT authentication — which and why? → [06](06-auth-and-security.md)
5. How do you protect against CSRF and XSS? → [06](06-auth-and-security.md)
6. WebSockets vs SSE vs polling? → [05](05-api-communication-patterns.md)
7. TCP vs UDP? → [04](04-backend-networking-nodejs.md)
8. What is a reverse proxy / load balancer, L4 vs L7? → [08](08-production-networking.md)
9. 502 vs 503 vs 504 — what causes each? → [09](09-troubleshooting-and-performance.md)
10. An API is slow — how do you debug it? → [09](09-troubleshooting-and-performance.md)

## Resources

- [MDN — HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- [Cloudflare Learning Center](https://www.cloudflare.com/learning/)
- [High Performance Browser Networking](https://hpbn.co/) — Ilya Grigorik (free online)
- *Computer Networking: A Top-Down Approach* — Kurose & Ross
