# 01 · How the Internet and Web Work

> **Subject:** Computer Networks (for full stack developers)
> **Status:** ✅ done
> **Last reviewed:** 2026-11-13

[← Back to Computer Networks](README.md) · [Questions for this topic →](questions/01-how-the-internet-works-qa.md)

## In one line

The internet is a huge network of computers that talk using agreed rules (protocols). Your browser (client) asks a server for something, the server answers. Everything else is detail about **how it finds the server** (IP, DNS) and **how they talk** (TCP, HTTP).

---

## 1. Client-server architecture

> 🎤 **Say it like this:** "In client-server, the client starts the conversation and asks for something, and the server listens and responds. The client is usually a browser or mobile app, the server is a process listening on a port. The server holds the data and the business logic, so many clients can share it."

- **Client** → starts the request (browser, mobile app, Postman, another backend).
- **Server** → always running, waiting on a port, sends a response.
- One machine can be both: your Node API is a *server* to the browser and a *client* to the database.

```
Browser / App  ──request──▶  Server (API)  ──query──▶  Database
     ▲                          │
     └────────response──────────┘
```

**Other model:** Peer-to-peer (P2P) — every node is both client and server (BitTorrent, WebRTC video calls).

---

## 2. Browser, server and internet

| Piece | What it does |
|---|---|
| **Browser** | Builds the request, does DNS lookup, opens connection, renders HTML/CSS/JS |
| **Internet** | The roads: ISPs, routers, cables that carry packets between networks |
| **Server** | A program (Node, Nginx, etc.) running on a machine, listening on an IP + port |

Data does not travel as one big piece. It is split into small **packets**; each packet can take a different path and is put back together at the other end (TCP handles this).

---

## 3. IP address: public vs private

> 🎤 **Say it like this:** "An IP address is the address of a device on a network. A public IP is reachable from the internet and is unique worldwide. A private IP only works inside a local network like your home Wi-Fi or an AWS VPC. Many private devices share one public IP using NAT."

| | Public IP | Private IP |
|---|---|---|
| Reachable from internet | ✅ Yes | ❌ No |
| Unique | Globally | Only inside its network |
| Given by | ISP / cloud provider | Router (DHCP) / VPC |
| Examples | `13.233.10.5` | `10.x.x.x`, `172.16–31.x.x`, `192.168.x.x` |

- **NAT (Network Address Translation):** your router swaps private IP → its public IP when going out, and back when the reply comes in. That is why 10 phones at home share one public IP.
- **IPv4** = `192.168.1.10` (32-bit, running out). **IPv6** = `2001:db8::1` (128-bit, huge space).

Deep dive (optional): [IP addressing & subnetting](deep-dives/ip-addressing-and-subnetting.md)

---

## 4. Ports and localhost

> 🎤 **Say it like this:** "IP finds the machine, the port finds the application on that machine. So `IP:port` identifies one exact process. localhost (127.0.0.1) is the loopback address — it means 'this same machine', the traffic never leaves your computer."

- Port range: `0–65535`. Below `1024` are "well-known" and need admin rights to bind.
- Common ports to remember:

| Port | Service |
|---|---|
| 80 | HTTP |
| 443 | HTTPS |
| 22 | SSH |
| 53 | DNS |
| 5432 | PostgreSQL |
| 3306 | MySQL |
| 27017 | MongoDB |
| 6379 | Redis |
| 3000 / 5173 / 8080 | Common dev servers |

- `localhost` / `127.0.0.1` → only your machine can reach it.
- `0.0.0.0` (when binding a server) → "listen on all network interfaces", so other machines / Docker can reach it.

> 💡 Classic bug: app works locally but not inside Docker → server is listening on `127.0.0.1` instead of `0.0.0.0`.

---

## 5. Domain names

A domain is a human-friendly name for an IP address.

```
https://api.shop.example.com:443/products?id=5
└─┬─┘   └┬┘ └┬─┘ └──┬──┘ └┬┘ └┬┘ └───┬──┘ └─┬─┘
scheme  sub  2nd   TLD  port path   query
    subdomain  level
```

- **TLD:** `.com`, `.in`, `.org`
- **Domain:** `example.com` (you buy this from a registrar like GoDaddy, Namecheap)
- **Subdomain:** `api.example.com`, `app.example.com` — free to create, you control them in DNS.

---

## 6. DNS resolution

> 🎤 **Say it like this:** "DNS is the phonebook of the internet — it converts a domain name into an IP address. The browser first checks its own cache, then the OS cache, then asks a recursive resolver (like your ISP or 8.8.8.8). If the resolver doesn't know, it asks root server → TLD server (.com) → authoritative server of that domain, which gives the final IP. The answer is cached based on its TTL."

```
Browser cache → OS cache (/etc/hosts too) → Recursive resolver
                                               │
                     ┌─────────────────────────┼─────────────────────────┐
                     ▼                         ▼                         ▼
               Root server  ──"ask .com"──▶  TLD server ──"ask ns1.example"──▶ Authoritative server
                                                                                  │
                                                              returns 93.184.216.34
```

**DNS record types (must know):**

| Record | Meaning | Example |
|---|---|---|
| **A** | Domain → IPv4 | `example.com → 93.184.216.34` |
| **AAAA** | Domain → IPv6 | |
| **CNAME** | Alias to another domain | `www → example.com` |
| **MX** | Mail server | |
| **TXT** | Text, used for verification (SPF, domain ownership) | |
| **NS** | Which servers are authoritative for this domain | |

- **TTL** = how long (seconds) the answer can be cached. Lower TTL before a migration so changes spread faster.
- DNS mostly uses **UDP port 53** (fast, small), TCP for large responses.

---

## 7. Request-response lifecycle — "What happens when you type a URL?"

> 🎤 **Say it like this (the famous question):**
> 1. Browser parses the URL and checks its cache.
> 2. **DNS lookup** turns the domain into an IP.
> 3. **TCP connection** is opened (3-way handshake: SYN, SYN-ACK, ACK).
> 4. **TLS handshake** for HTTPS — agree on encryption, verify certificate.
> 5. Browser sends the **HTTP request** (method, path, headers, cookies).
> 6. Request goes through CDN / load balancer / reverse proxy to the **server**, which runs the logic, maybe hits DB or cache.
> 7. Server sends the **HTTP response** (status, headers, body).
> 8. Browser **parses HTML**, builds the DOM, downloads CSS/JS/images (more requests), builds CSSOM, runs JS, then layout and paint.
> 9. Connection is kept alive for reuse.

That one answer touches DNS, TCP, TLS, HTTP and rendering — interviewers love it because they can go deep on any step.

---

## 8. OSI and TCP/IP models (conceptual)

> 🎤 **Say it like this:** "These are layered models that split networking into responsibilities. OSI has 7 layers and is mostly for theory and vocabulary. TCP/IP has 4 layers and is what the internet actually runs on. As a developer I mostly work at the application layer (HTTP), and I care about the transport layer (TCP/UDP) and the network layer (IP)."

| TCP/IP layer | OSI layers | What it handles | Examples | Developer cares because… |
|---|---|---|---|---|
| Application | 7, 6, 5 | App data format | HTTP, DNS, TLS, WebSocket | This is where you write code |
| Transport | 4 | Ports, reliability | TCP, UDP | Timeouts, connections, keep-alive |
| Internet | 3 | IP addresses, routing | IP, ICMP (ping) | IPs, VPCs, subnets |
| Link | 2, 1 | Physical delivery | Ethernet, Wi-Fi | Rarely touch it |

**Where it's useful for you:**
- "L4 vs L7 load balancer" = transport level (IP+port) vs application level (HTTP path/headers).
- Debugging: "is it DNS (L7)? Is the port open (L4)? Can I ping the host (L3)?"

Deep dive (optional): [OSI & TCP/IP models](deep-dives/osi-and-tcp-ip-models.md)

---

## ⚡ Quick revision

- Client asks, server answers. One service can be both.
- **IP** = which machine, **port** = which app, **DNS** = name → IP.
- Private IPs live inside networks; NAT maps them to a public IP.
- `127.0.0.1` = only me; `0.0.0.0` = listen on all interfaces.
- DNS: browser → OS → resolver → root → TLD → authoritative; cached by TTL.
- URL flow: DNS → TCP → TLS → HTTP request → server → response → render.
- OSI 7 layers (theory), TCP/IP 4 layers (real).
