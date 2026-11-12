# Q&A · Networking Troubleshooting and Performance

[← Back to Computer Networks](../README.md) · [Read the notes →](../09-troubleshooting-and-performance.md)

> Short answers you can say out loud. 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

### 🟢 Q1. Connection refused vs connection timeout?
With refused, the host answered, but nothing is listening on that port: the service is down, the port is wrong, or the app is bound to localhost. With a timeout, no answer came back at all. Usually a firewall or security group is dropping packets, the IP is wrong, or the host is down.

### 🟡 Q2. What does ECONNRESET mean?
The connection was open but the other side abruptly closed it. Common causes: the server crashed, or a proxy or LB idle timeout closed it. It also happens when the client reuses a keep-alive connection that the server just closed.

### 🟢 Q3. What does ENOTFOUND mean?
The DNS lookup failed. The hostname has a typo, the record doesn't exist, or it's an internal name used from outside the network.

### 🟢 Q4. Difference between 502, 503 and 504?
All three come from a proxy or LB. 502 means the upstream sent a bad response or none, often because the app crashed. 503 means no healthy backend is available or the service is overloaded. 504 means the upstream took too long and the proxy timed out.

### 🔴 Q5. Users get random 502s from an ALB in front of Node. Cause?
The usual cause is a keep-alive mismatch. Node's default `keepAliveTimeout` is 5s and the ALB idle timeout is 60s. Node closes an idle connection just as the ALB reuses it. Set `keepAliveTimeout` above 60s and `headersTimeout` higher still. Also check for app crashes or OOM restarts.

### 🟡 Q6. Why does an API work in Postman but give a CORS error in the browser?
CORS is enforced only by browsers. The server isn't returning the right `Access-Control-Allow-*` headers for that origin, or the preflight OPTIONS request fails. Often the request did reach the server.

### 🟡 Q7. How do you debug a CORS error?
1. Check the OPTIONS preflight in the Network tab.
2. Confirm `Allow-Origin` matches the exact origin.
3. With cookies, the origin can't be `*` and you need `Allow-Credentials: true`.
4. Make sure custom headers and methods are allowed.
5. Check whether a 500 or 404 is hiding behind it.

### 🔴 Q8. An API endpoint is slow. How do you investigate?
In the Network tab, compare TTFB with download time. High TTFB means the backend is slow. Then use logs or APM traces to find the slow part: a DB query (N+1, missing index, check with `EXPLAIN`), a sequential external call, pool waits, or event loop blocking. If the download is long, cut the payload with pagination, compression or fewer fields.

### 🟢 Q9. Latency vs bandwidth vs throughput?
Latency is the time for one trip, in ms. Bandwidth is the maximum capacity of the pipe. Throughput is the data that actually gets through. For APIs, latency and the number of round trips usually matter most.

### 🟡 Q10. How do you reduce network latency for users?
Use a CDN or edge caching and deploy closer to users. Reuse connections with keep-alive and HTTP/2/3, and cut round trips through batching, fewer redirects and `preconnect`. Cache aggressively and compress payloads.

### 🟡 Q11. What is packet loss and how does it affect TCP?
Some packets don't arrive. TCP re-sends them and slows its sending rate, so even small loss makes requests noticeably slower. A lost packet also blocks the data behind it (head-of-line blocking).

### 🟡 Q12. What is exponential backoff with jitter?
Wait times between retries grow exponentially (200ms, 400ms, 800ms…) up to a cap, and each one is randomized. The random part stops many clients from retrying at the same instant and overloading a recovering service.

### 🟡 Q13. Which requests are safe to retry?
Retry only transient failures (timeouts, resets, 502, 503, 504, 429) on idempotent operations like GET, PUT and DELETE. Don't retry POST or payments unless you send an idempotency key. Never retry 4xx validation or auth errors.

### 🟡 Q14. What is a circuit breaker?
It stops calls to a dependency that keeps failing. **Closed** means calls flow normally. **Open** comes after too many failures: it fails fast and returns a fallback. **Half-open** comes after a cooldown: a few test calls go through, and success closes it again. This prevents cascading failures.

### 🟡 Q15. Why must every network call have a timeout?
Without one, a slow dependency holds connections and memory open forever. Pools fill up and the whole service hangs. Inner timeouts (DB, external APIs) must be shorter than outer ones (proxy, client).

### 🔴 Q16. Under load, requests start hanging with "timeout acquiring connection". What's happening?
The DB connection pool is exhausted. Likely causes: connections leaked by not releasing on error, slow queries holding connections, or pool size × instances exceeding the DB's max connections. Fix it by releasing in `finally`, adding query timeouts, fixing slow queries, and using PgBouncer or RDS Proxy.

### 🟡 Q17. What is a thundering herd problem?
Many requests hit the same resource at the same moment, e.g. a popular cache key expires and everything goes to the DB. Fix it with jittered TTLs, request coalescing or locks, stale-while-revalidate, and jittered reconnects.

### 🟢 Q18. How do you check which process is using a port?
`lsof -i :3000` on macOS/Linux, or `ss -tulpn`/`netstat -an`. Kill the old process if you get `EADDRINUSE`.

### 🟡 Q19. How do you measure where time goes in a request using curl?
`curl -o /dev/null -s -w "dns:%{time_namelookup} connect:%{time_connect} tls:%{time_appconnect} ttfb:%{time_starttransfer} total:%{time_total}" URL`. This splits the time into DNS, TCP, TLS, server wait and total.

### 🟡 Q20. How do you check if a remote port is open, and if a TLS cert is valid?
`nc -zv host 5432` checks whether the port is open. `openssl s_client -connect host:443 -servername host` shows the certificate chain, expiry and handshake errors.

### 🔴 Q21. A service in a private subnet can't reach a third-party API (timeouts). What do you check?
Check that the route table sends `0.0.0.0/0` to a NAT Gateway and that the NAT sits in a public subnet with an Internet Gateway. Check that security group outbound rules and NACLs allow the traffic, both directions for NACLs. Check that DNS resolves, and that the third party isn't IP-allowlisting the NAT's Elastic IP.

### 🟢 Q22. When would you use Wireshark?
When logs and curl don't explain the problem: TLS handshake failures, connection resets, retransmissions or odd timeouts. Filter with `dns`, `http`, `tcp.port == 443` or `tcp.flags.reset == 1`.
