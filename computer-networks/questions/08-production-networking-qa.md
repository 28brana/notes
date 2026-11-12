# Q&A · Production Architecture and Deployment Networking

[← Back to Computer Networks](../README.md) · [Read the notes →](../08-production-networking.md)

> Short answers you can say out loud. 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

### 🟢 Q1. Forward proxy vs reverse proxy?
A forward proxy acts for clients and hides them (corporate proxy, VPN). A reverse proxy acts for servers and hides them (Nginx in front of Node). Clients talk to the reverse proxy and never see the backend servers.

### 🟢 Q2. Why put Nginx in front of a Node.js app?
It handles TLS, gzip, static files, slow clients and load balancing much more efficiently, and lets only ports 80/443 be public. Node then focuses on business logic.

### 🟡 Q3. What headers should a reverse proxy pass to the app, and why?
`Host`, `X-Forwarded-For` (real client IP), `X-Forwarded-Proto` (http or https) and `X-Real-IP`. Without them the app sees the proxy's IP and thinks every request is HTTP. In Express, enable `app.set('trust proxy', 1)`.

### 🟡 Q4. How do you proxy WebSockets through Nginx?
Use HTTP/1.1 and forward the upgrade headers: `proxy_http_version 1.1; proxy_set_header Upgrade $http_upgrade; proxy_set_header Connection "upgrade";`. Also raise `proxy_read_timeout`, or idle sockets get closed at 60s.

### 🟢 Q5. What is a load balancer and why use it?
It spreads traffic across multiple servers, removes unhealthy ones with health checks, and enables horizontal scaling and high availability.

### 🟡 Q6. Name some load balancing algorithms.
Round robin (in turn), weighted round robin (by capacity), least connections (fewest active — good for long requests), IP hash (same client → same server).

### 🟡 Q7. What are sticky sessions and why avoid them?
The LB sends a user to the same server every time, usually via a cookie, because session data is in that server's memory. It causes uneven load and lost sessions when a server dies. Better to keep servers stateless with Redis sessions or JWT.

### 🟡 Q8. L4 vs L7 load balancer?
L4 sees only IP and port and forwards TCP/UDP — very fast (AWS NLB). L7 understands HTTP and can route by path/host/header, terminate TLS and set cookies (AWS ALB). Web apps usually use L7.

### 🟡 Q9. What is TLS termination?
Decrypting HTTPS at the load balancer or Nginx, then sending plain HTTP inside the private network. Certificates live in one place and app servers skip the crypto work. Re-encrypt to the backend if compliance requires it.

### 🟢 Q10. What is a CDN and how does it help?
A global network of edge servers that cache content close to users. Lower latency, less origin load, better handling of traffic spikes and DDoS.

### 🟡 Q11. How do you handle CDN cache invalidation on deploy?
Use content-hashed file names (`main.a1b2c3.js`) cached for a year, and keep `index.html` on `no-cache`. A new deploy produces new names, so no purge is needed. Use manual invalidation only for fixed URLs.

### 🟡 Q12. How do you point a root domain to an AWS load balancer?
Use a Route 53 ALIAS A record pointing at the ALB's DNS name. A root domain can't be a CNAME, and ALB IPs change, so you never hardcode an IP.

### 🟢 Q13. Let's Encrypt vs AWS ACM?
Both are free. Let's Encrypt (via certbot) gives 90-day certs you install on your own server, such as Nginx, with auto renew. ACM certs auto-renew but can only be attached to AWS services like ALB, CloudFront and API Gateway. You can't export the key.

### 🟢 Q14. What is a VPC?
Your own isolated private network in AWS, with an IP range you choose (e.g. `10.0.0.0/16`) split into subnets across availability zones.

### 🟡 Q15. Public vs private subnet?
A public subnet routes `0.0.0.0/0` to an Internet Gateway, which suits ALBs and NAT gateways. A private subnet has no direct internet route, so it's where app servers and databases go. A private subnet reaches out only via a NAT Gateway.

### 🟡 Q16. What does a NAT Gateway do?
It lets resources in private subnets make outbound internet calls (APIs, package installs) while blocking any inbound connections from the internet. It sits in a public subnet.

### 🟡 Q17. Security group vs NACL?
A security group is a stateful firewall at the instance level with allow rules only. Return traffic is allowed automatically. A NACL is a stateless firewall at the subnet level with allow and deny rules, so you must allow both directions.

### 🟡 Q18. What is an API Gateway?
A single entry point for APIs that handles auth, rate limiting, API keys, validation, CORS, caching and routing to services or Lambdas, so each service doesn't have to.

### 🟡 Q19. In Docker Compose, why can't my API connect to `localhost:5432` for Postgres?
Inside a container, `localhost` means the container itself. Use the service name as the host, e.g. `db:5432`. Compose puts all services on a shared network with DNS by service name.

### 🔴 Q20. Design a secure network setup for a Node + Postgres app on AWS.
Route 53 → CloudFront (S3 frontend) and ALB in public subnets with an ACM cert. Node on ECS/EC2 in private subnets, RDS in private subnets. Chain the security groups: ALB allows 443 from anywhere, app allows 3000 only from the ALB SG, DB allows 5432 only from the app SG. Outbound goes through a NAT Gateway. Use Multi-AZ for availability.

### 🔴 Q21. After deploying behind an ALB, users randomly get logged out. Why?
Sessions are probably in server memory and requests hit different instances. Move sessions to Redis or use JWT. Turning on sticky sessions is the quick fix. Also check that the cookie `Secure` flag works, which needs `trust proxy` for `X-Forwarded-Proto`.

### 🔴 Q22. Your site shows the old frontend after deploy for some users. What do you check?
Check whether `index.html` is cached at the CDN or browser. Set it to `no-cache` and invalidate it. Confirm bundles have hashed names. Check service worker caching and DNS TTL if the origin changed.
