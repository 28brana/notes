# 08 · Production Architecture and Deployment Networking

> **Subject:** Computer Networks (for full stack developers)
> **Status:** ✅ done
> **Last reviewed:** 2026-11-13

[← Back to Computer Networks](README.md) · [Questions for this topic →](questions/08-production-networking-qa.md)

## In one line

In production, a request does not hit your Node app directly. It goes **DNS → CDN → load balancer → reverse proxy → app (private network) → database (private network)**. Each hop exists for speed, security or scale.

---

## 1. Forward proxy vs reverse proxy

> 🎤 **Say it like this:** "A forward proxy sits in front of clients and talks to the internet on their behalf — like a corporate proxy or VPN. A reverse proxy sits in front of servers and receives requests on their behalf — like Nginx in front of Node. The client doesn't know which backend server actually answered."

```
Forward proxy:   [Clients] ──▶ Proxy ──▶ Internet ──▶ Servers     (hides the client)
Reverse proxy:   Clients ──▶ Internet ──▶ Proxy ──▶ [Servers]     (hides the server)
```

| | Forward proxy | Reverse proxy |
|---|---|---|
| Works for | Clients | Servers |
| Hides | Client identity | Backend servers |
| Use | Content filtering, corporate access, VPN | TLS termination, load balancing, caching, compression |
| Examples | Squid, corporate proxy | Nginx, HAProxy, Traefik, Envoy |

---

## 2. Reverse proxy and Nginx basics

> 🎤 **Say it like this:** "I put Nginx in front of Node because it handles TLS, gzip, static files and slow clients very efficiently, and forwards only the real API work to Node. It also lets me run multiple apps on one server behind ports 80/443 and add rate limiting or caching without touching app code."

**Why not expose Node directly?**
- TLS termination and certificates in one place.
- Serves static files much faster than Node.
- Buffers slow clients so Node isn't blocked.
- Load balances between multiple Node processes.
- Hides internal ports (3000) — only 80/443 are public.

**A small realistic config:**

```nginx
upstream node_app {
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;          # 2 Node processes → load balanced
    keepalive 32;                    # reuse connections to Node
}

server {
    listen 80;
    server_name api.example.com;
    return 301 https://$host$request_uri;   # force HTTPS
}

server {
    listen 443 ssl http2;
    server_name api.example.com;

    ssl_certificate     /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;

    gzip on;
    gzip_types application/json text/css application/javascript;

    client_max_body_size 10m;        # max upload size

    location /static/ {
        root /var/www/app;           # Nginx serves files, Node never sees it
        expires 30d;
    }

    location /socket.io/ {           # WebSocket upgrade
        proxy_pass http://node_app;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }

    location / {
        proxy_pass http://node_app;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 60s;
    }
}
```

> 💡 Behind a proxy, `req.ip` in Express shows the proxy's IP. Use `app.set('trust proxy', 1)` so Express reads `X-Forwarded-For` and `X-Forwarded-Proto` (needed for real client IP, rate limiting, and `secure` cookies).

---

## 3. Load balancer

> 🎤 **Say it like this:** "A load balancer spreads incoming traffic across multiple servers so no single server is overloaded, and it removes unhealthy servers using health checks. It gives us horizontal scaling and high availability — if one instance dies, users don't notice."

**Algorithms:**

| Algorithm | How | Good for |
|---|---|---|
| Round robin | 1 → 2 → 3 → 1… | Similar servers, similar requests |
| Weighted round robin | Bigger server gets more | Mixed instance sizes |
| Least connections | Send to server with fewest active connections | Long or uneven requests (WebSocket, uploads) |
| IP hash | Same client IP → same server | Basic stickiness |

**Health checks:** LB calls something like `GET /health` every few seconds. Fail N times → server removed. Pass again → added back.

```js
app.get('/health', (req, res) => res.status(200).send('ok'));       // liveness: process is up
app.get('/ready', async (req, res) => {                               // readiness: dependencies OK
  try { await db.query('SELECT 1'); res.send('ready'); }
  catch { res.status(503).send('db down'); }
});
```

**Sticky sessions:** LB sends the same user to the same server (using a cookie). Needed when session is stored in server memory. Downside: uneven load, and the user loses session if that server dies.
👉 Better: make servers **stateless** — store sessions in Redis or use JWT. Then any server can handle any request.

---

## 4. L4 vs L7 load balancing (conceptual)

> 🎤 **Say it like this:** "An L4 load balancer works at the transport level — it only sees IP and port and forwards TCP/UDP connections, so it's very fast. An L7 load balancer understands HTTP — it can route by path, host or headers, terminate TLS and add cookies. In AWS, NLB is L4 and ALB is L7."

| | L4 (Transport) | L7 (Application) |
|---|---|---|
| Sees | IP, port, TCP/UDP | URL, host, headers, cookies, method |
| Routing | By IP + port only | `/api` → service A, `/admin` → service B |
| TLS | Usually passes through (can terminate) | Terminates TLS |
| Speed | Faster, less work | Slightly more overhead |
| Extras | Static IP, millions of conn/sec | Sticky cookies, redirects, auth, WAF |
| AWS | **NLB** | **ALB** |
| Use | Raw TCP, gaming, very high throughput, gRPC passthrough | Web apps, REST APIs, microservices |

---

## 5. TLS termination and SSL certificates

> 🎤 **Say it like this:** "TLS termination means HTTPS is decrypted at the load balancer or Nginx, and traffic inside the private network goes as plain HTTP (or re-encrypted if compliance needs it). This keeps certificates in one place and removes crypto work from the app servers."

```
Browser ══HTTPS══▶ ALB / Nginx (decrypts here) ──HTTP──▶ Node apps (private subnet)
```

**Getting certificates:**

| Option | How | Where used |
|---|---|---|
| **Let's Encrypt + certbot** | Free, 90-day certs, auto renew via cron | Nginx on your own VM |
| **AWS ACM** | Free, auto-renewing, can't download the private key | ALB, CloudFront, API Gateway |
| Paid CA | Buy (DigiCert etc.), EV/OV certs | Enterprise requirements |

```bash
sudo certbot --nginx -d example.com -d api.example.com   # gets cert + edits nginx config
sudo certbot renew --dry-run                              # test auto-renew
```

- Cert must match the domain. `*.example.com` (wildcard) covers `api.` and `app.` but **not** `example.com` itself or `a.b.example.com`.
- ACM certs for **CloudFront must be in `us-east-1`**.
- Common outage: certificate expired because auto-renew silently failed → set alerts.

---

## 6. CDN

> 🎤 **Say it like this:** "A CDN is a network of servers spread around the world that caches content close to users. A user in Mumbai gets the file from a Mumbai edge server instead of my origin in the US, so it's much faster and my origin gets less load. It's used for static assets, images, videos, and even cached API responses. It also helps absorb DDoS."

```
User (Mumbai) ──▶ Edge (Mumbai) ── cache HIT ──▶ return instantly
                         │
                     cache MISS
                         ▼
                 Origin (S3 / ALB in us-east-1) → response cached at edge for TTL
```

- Caching is controlled by `Cache-Control` headers from origin (see [Browser networking](03-browser-networking.md)).
- **Cache invalidation** (the hard part):
  - Best: **versioned file names** — `app.3f9a2c.js`. New deploy = new file name = no invalidation needed. Cache these for 1 year (`immutable`).
  - Keep `index.html` short-cached / `no-cache` so it always points to the latest bundles.
  - Manual purge: `aws cloudfront create-invalidation --paths "/*"` (slow-ish, and costs after free limit).
- Examples: CloudFront, Cloudflare, Akamai, Fastly.

---

## 7. DNS configuration for a real app

> 🎤 **Say it like this:** "I point the root domain to the load balancer or CloudFront using an ALIAS record in Route 53, because load balancer IPs change. Subdomains like `www` or `app` are CNAMEs. I keep TTL low before migrations and verify domain ownership with TXT records."

| Record | Name | Points to | Why |
|---|---|---|---|
| A (ALIAS) | `example.com` | CloudFront distribution | Root can't be a CNAME; ALIAS solves it |
| CNAME | `www.example.com` | `example.com` | Alias |
| A (ALIAS) | `api.example.com` | ALB DNS name | ALB IPs keep changing — never hardcode them |
| CNAME | `app.example.com` | `myapp.vercel.app` | Hosted frontend |
| TXT | `example.com` | `google-site-verification=…` | Ownership / SPF |
| MX | `example.com` | Mail provider | Email |

> 💡 Never put an ALB's IP in an A record — AWS changes them. Always ALIAS/CNAME to its DNS name.

---

## 8. Domain and subdomain routing

Two common ways to send traffic to different services:

```
Host-based (subdomain):              Path-based:
app.example.com   → React (S3/CDN)   example.com/        → frontend
api.example.com   → Node API         example.com/api/*   → API service
admin.example.com → Admin app        example.com/auth/*  → auth service
```

- Done in Nginx (`server_name`, `location`), ALB listener rules, or an Ingress in Kubernetes.
- **Same domain with path routing → no CORS needed** (same origin). Subdomains = different origins → need CORS and careful cookie `Domain` settings.

---

## 9. AWS networking basics

### Full picture

```
                      Route 53 (DNS)
                            │
                      CloudFront (CDN, TLS, WAF)
                     ╱                ╲
          S3 (React build)      ┌─────────────── VPC 10.0.0.0/16 ───────────────┐
                                │                                                │
                                │  Public subnet 10.0.1.0/24                     │
                                │   ├─ ALB (L7, TLS via ACM)                     │
                                │   └─ NAT Gateway ──▶ Internet Gateway ──▶ 🌐   │
                                │          ▲                                     │
                                │  Private subnet 10.0.2.0/24                    │
                                │   └─ EC2 / ECS (Node API) — no public IP       │
                                │          │                                     │
                                │  Private subnet 10.0.3.0/24                    │
                                │   └─ RDS / ElastiCache (Redis)                 │
                                └────────────────────────────────────────────────┘
```

### VPC

> 🎤 **Say it like this:** "A VPC is my own private, isolated network inside AWS. I choose its IP range, like `10.0.0.0/16`, and split it into subnets across availability zones. Only resources I explicitly expose are reachable from the internet."

**CIDR in 10 seconds:** the `/number` says how many bits are fixed. Smaller number = bigger network.
- `10.0.0.0/16` → ~65,000 IPs (the whole VPC)
- `10.0.1.0/24` → 256 IPs (one subnet; AWS reserves 5)

### Public vs private subnet

| | Public subnet | Private subnet |
|---|---|---|
| Route table | `0.0.0.0/0 → Internet Gateway` | `0.0.0.0/0 → NAT Gateway` (or nothing) |
| Reachable from internet | Yes (if it has a public IP) | No |
| Put here | ALB, NAT Gateway, bastion host | App servers, databases, Redis |

### Internet Gateway vs NAT Gateway

- **Internet Gateway:** two-way door between VPC and internet for public subnets.
- **NAT Gateway:** **outbound-only** door for private subnets. App servers can call Stripe or `npm install`, but nobody from outside can start a connection to them. Lives in a public subnet. Costs money per hour + per GB — a common surprise on the bill.

### Security groups vs NACLs

> 🎤 **Say it like this:** "Security groups are stateful firewalls attached to instances — if I allow inbound traffic, the reply is automatically allowed. NACLs are stateless firewalls at the subnet level — I must allow both directions. Day to day I mostly use security groups and reference other security groups instead of IPs."

| | Security Group | NACL |
|---|---|---|
| Level | Instance / ENI | Subnet |
| State | **Stateful** (return traffic auto-allowed) | **Stateless** (must allow return ports too) |
| Rules | Allow only | Allow and deny |
| Typical use | Main firewall | Extra layer, block bad IP ranges |

**Classic secure setup (chain of security groups):**
```
alb-sg : inbound 443 from 0.0.0.0/0
app-sg : inbound 3000 from alb-sg only
db-sg  : inbound 5432 from app-sg only
```

### API Gateway

> 🎤 **Say it like this:** "An API Gateway is a single entry point in front of my APIs or Lambdas. It handles cross-cutting things — authentication, rate limiting/throttling, API keys, request validation, CORS, caching and routing to different services — so each service doesn't implement them itself."

- AWS API Gateway is common with **Lambda** (serverless). With containers, ALB is often enough.
- Others: Kong, Apigee, Nginx used as gateway.

### CloudFront and ALB (quick)

- **CloudFront** = AWS CDN. Serves S3 frontend, caches static files, can sit in front of ALB, terminates TLS at the edge, integrates WAF.
- **ALB** = AWS L7 load balancer. Path/host routing, health checks, TLS with ACM, target groups (EC2, ECS, Lambda, IPs), WebSocket supported.

---

## 10. Docker and Kubernetes networking (basics)

> 🎤 **Say it like this:** "Each container has its own network namespace. With `-p 8080:3000` I map host port 8080 to container port 3000. In Docker Compose, all services share a network and reach each other by service name, like `postgres:5432` — not localhost, because localhost inside a container is the container itself."

```bash
docker run -p 8080:3000 my-api        # host:8080 → container:3000
```

```yaml
# docker-compose.yml
services:
  api:
    build: .
    ports: ["3000:3000"]
    environment:
      DATABASE_URL: postgres://user:pass@db:5432/app   # "db" = service name
  db:
    image: postgres:16                                # no ports exposed = only reachable inside network
```

- Default network type: **bridge** (private network on the host).
- App inside container must listen on `0.0.0.0`, not `127.0.0.1`.
- **Kubernetes:** a **Service** gives a stable name/IP to a set of pods (internal load balancing); an **Ingress** is the L7 router that brings outside HTTP traffic in by host/path (usually Nginx or ALB controller).

---

## ⚡ Quick revision

- Forward proxy hides clients; reverse proxy (Nginx) hides servers.
- Nginx: TLS, gzip, static files, load balancing, WebSocket upgrade, `X-Forwarded-*` headers → `trust proxy` in Express.
- LB algorithms: round robin, least connections, IP hash. Health checks remove bad servers.
- Prefer stateless servers (Redis/JWT) over sticky sessions.
- L4 = IP+port (NLB, fast). L7 = HTTP aware (ALB, path/host routing).
- TLS terminates at LB/Nginx; certs from Let's Encrypt or ACM (CloudFront certs in us-east-1).
- CDN = edge caching; use hashed file names instead of invalidations.
- Root domain → ALIAS; subdomains → CNAME; never hardcode ALB IPs.
- VPC: public subnet (IGW) for ALB/NAT; private subnet for apps/DB; NAT = outbound only.
- Security group = stateful, instance level. NACL = stateless, subnet level.
- Docker: containers talk by service name; listen on `0.0.0.0`.
