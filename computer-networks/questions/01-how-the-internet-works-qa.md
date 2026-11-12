# Q&A · How the Internet and Web Work

[← Back to Computer Networks](../README.md) · [Read the notes →](../01-how-the-internet-works.md)

> Short answers you can say out loud. 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

### 🟢 Q1. What is client-server architecture?
The client starts the request, the server listens and responds. The server holds shared data and logic so many clients can use it. A backend can be a server to the browser and a client to the database at the same time.

### 🟢 Q2. Difference between public and private IP?
Public IP is unique on the internet and reachable from anywhere. Private IP (`10.x`, `172.16–31.x`, `192.168.x`) works only inside a local network or VPC. Private devices reach the internet through NAT, which maps them to a public IP.

### 🟢 Q3. What is a port? Why do we need it?
IP identifies the machine, port identifies the application on it. One server can run Nginx on 443, Postgres on 5432 and Redis on 6379 — ports tell the OS which process gets the traffic.

### 🟢 Q4. What is localhost?
`localhost` = `127.0.0.1`, the loopback address. Traffic never leaves the machine. Used for local development.

### 🟡 Q5. Difference between binding a server to `127.0.0.1` and `0.0.0.0`?
`127.0.0.1` accepts connections only from the same machine. `0.0.0.0` listens on all network interfaces, so other machines or Docker host can reach it. If an app in Docker is "not reachable", this is the first thing to check.

### 🟢 Q6. What is DNS?
The phonebook of the internet — it converts a domain name like `google.com` into an IP address.

### 🟡 Q7. Explain DNS resolution step by step.
Browser cache → OS cache (and `/etc/hosts`) → recursive resolver (ISP / 8.8.8.8). If not cached, the resolver asks a root server → TLD server (`.com`) → the domain's authoritative server, which returns the IP. Result is cached for the TTL.

### 🟡 Q8. A record vs CNAME?
A record maps a name directly to an IPv4 address. CNAME maps a name to another name (alias), e.g. `www.example.com → example.com`. You can't put a CNAME on the root domain in standard DNS (many providers offer "ALIAS/ANAME" for that).

### 🟡 Q9. What is DNS TTL and why does it matter?
TTL is how long resolvers cache the answer. High TTL = fewer lookups but slow changes. Before migrating a server, lower the TTL so the new IP spreads quickly.

### 🔴 Q10. What happens when you type a URL in the browser and press Enter?
1. Check browser cache. 2. DNS lookup → IP. 3. TCP 3-way handshake. 4. TLS handshake (HTTPS). 5. Send HTTP request with headers/cookies. 6. Passes CDN / load balancer → server runs logic, DB/cache. 7. Server returns response. 8. Browser parses HTML, fetches CSS/JS/images, builds DOM + CSSOM, runs JS, layout and paint. Connection is kept alive for reuse.

### 🟡 Q11. Why does DNS use UDP?
DNS queries are small and need to be fast. UDP has no handshake, so one packet out, one back. It falls back to TCP for large responses (and zone transfers).

### 🟢 Q12. What is the OSI model? Name the layers.
A 7-layer conceptual model: Physical, Data Link, Network, Transport, Session, Presentation, Application. Mnemonic: *Please Do Not Throw Sausage Pizza Away*.

### 🟡 Q13. OSI vs TCP/IP model?
OSI = 7 layers, theoretical reference. TCP/IP = 4 layers (Application, Transport, Internet, Link) and is what the internet actually uses. OSI is still used as vocabulary — "L4", "L7".

### 🟡 Q14. At which layer do HTTP, TCP and IP work?
HTTP → Application (L7). TCP/UDP → Transport (L4). IP → Network (L3).

### 🟡 Q15. What is NAT?
Network Address Translation. The router replaces the private source IP with its public IP on outgoing traffic and maps replies back. Lets many devices share one public IP and hides internal IPs. In AWS, a NAT Gateway does this for private subnets.

### 🔴 Q16. A website works for you but not for a colleague in another city. What could it be?
DNS (their resolver still caches the old IP — TTL), CDN edge issue in their region, geo-blocking / firewall rules, their ISP blocking, or a corporate proxy. Check with `nslookup` / `dig` from their machine and compare IPs.

### 🟢 Q17. IPv4 vs IPv6?
IPv4 is 32-bit (~4.3 billion addresses, running out), written like `192.168.1.1`. IPv6 is 128-bit (practically unlimited), written like `2001:db8::1`, and reduces the need for NAT.
