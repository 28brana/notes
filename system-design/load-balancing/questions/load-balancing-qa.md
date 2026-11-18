# Q&A · Load Balancing

[← Back to Load Balancing](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 01 · Load Balancers

[Read the notes →](../01-load-balancers.md)

> [!question]- 🟢 Q1. What is a load balancer and why do we need one?
> It sits in front of a group of servers and gives clients one address. It spreads requests across the servers and skips any that fail health checks. That gives us horizontal scaling, higher availability, and zero-downtime deploys.

> [!question]- 🟡 Q2. What's the difference between an L4 and an L7 load balancer?
> L4 only sees IPs and ports, so it's very fast but can't look inside the request. L7 reads the HTTP request, so it can route by path, host, or cookie, terminate TLS, and retry — but it uses more CPU. Many setups use L4 at the edge and L7 behind it.

> [!question]- 🟡 Q3. What are sticky sessions and why should you usually avoid them?
> Sticky sessions send the same user to the same server, usually via a cookie, because that server holds their session in memory. They cause uneven load and users lose their session if that server dies. The better fix is stateless servers with sessions in Redis or in a token.

> [!question]- 🔴 Q4. Your load balancer is the only entry point. How do you stop it from being a single point of failure?
> Run at least two. A common setup is an active-passive pair sharing a floating IP, so the standby takes over in seconds. At bigger scale, run several LBs active-active behind DNS or anycast, or use a managed cloud LB that's already spread across zones.

---

## 02 · Load Balancing Algorithms

[Read the notes →](../02-load-balancing-algorithms.md)

> [!question]- 🟢 Q5. Explain round robin and weighted round robin.
> Round robin sends requests to servers in a fixed rotation. Weighted round robin gives bigger servers a bigger share, so a weight-3 server gets three requests for every one a weight-1 server gets. Both ignore live load, so they work best when servers and requests are similar.

> [!question]- 🟡 Q6. When would you choose least connections over round robin?
> When requests take very different amounts of time, like quick API calls mixed with long uploads, or long-lived WebSocket connections. Round robin would keep piling work onto a server that's still busy. Least connections sends new work to whoever has the fewest active connections.

> [!question]- 🟡 Q7. What is "power of two choices" and why does it work well?
> Pick two servers at random and send the request to the less loaded one. It gets close to perfect balance without checking every server. It also avoids the herd effect, where many load balancers all choose the same "least loaded" server at once.

> [!question]- 🔴 Q8. You added one cache server and the hit rate crashed. Requests are routed with hash of key mod N. Why, and how do you fix it?
> With mod N, changing N from 4 to 5 remaps almost every key to a different server, so nearly the whole cache misses at once. Switch to consistent hashing, where adding a server only moves about 1/N of the keys. Virtual nodes keep the load even.

---

## 03 · DNS Load Balancing

[Read the notes →](../03-dns-load-balancing.md)

> [!question]- 🟢 Q9. How does DNS load balancing work?
> The DNS server returns different IP addresses for the same domain — in rotation, by weight, by the user's location, or skipping unhealthy ones. It's usually used to pick a region, and a normal load balancer inside that region picks the server.

> [!question]- 🔴 Q10. A region went down and you removed it from DNS, but users still hit it for 20 minutes. Why?
> DNS answers are cached by browsers, OSes, and resolvers for the TTL, and some ignore the TTL and cache longer. Until those caches expire, clients keep using the old IP. Use low TTLs like 60 seconds for failover records, and consider anycast or a global LB for faster switching.

---

## 04 · Anycast Routing

[Read the notes →](../04-anycast-routing.md)

> [!question]- 🟡 Q11. What is anycast and where is it used?
> Many locations announce the same IP address, and internet routing sends each user to the nearest one. If a site fails it stops announcing the route, so traffic moves without waiting for DNS TTLs. It's used by CDNs, public DNS like 1.1.1.1, and DDoS protection.
