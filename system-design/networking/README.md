# System Design · Networking

[← Back to System Design](../README.md) · [Interview Q&A →](questions/networking-qa.md) · [Practice questions 📝](questions/practice-questions.md)

How requests actually travel from a user to your servers. Interviewers expect you to know L4 vs L7, TCP vs UDP, what HTTP/2/3 change, how DNS caching affects failover, and why a reverse proxy sits in front of everything.

| # | Topic | In one line | Priority |
|---|---|---|---|
| 01 | [OSI Model](01-osi-model.md) | 7 layers; what matters is L4 (TCP/UDP) vs L7 (HTTP) | ⚡ Medium |
| 02 | [IP Address](02-ip-address.md) | Public vs private IPs, NAT, CIDR, never hard-code IPs | ⚡ Medium |
| 03 | [TCP vs UDP](03-tcp-vs-udp.md) | Reliable and ordered vs fast and fire-and-forget | 🔥 High |
| 04 | [HTTP & HTTPS](04-http-https.md) | Stateless requests, safe retries, HTTP/2/3, TLS | 🔥 High |
| 05 | [DNS](05-dns.md) | Name → IP, caching and TTL, GeoDNS and failover | 🔥 High |
| 06 | [Proxy vs Reverse Proxy](06-proxy-vs-reverse-proxy.md) | Acts for clients vs servers; LB, TLS, cache at the edge | 🔥 High |
| 07 | [Checksums](07-checksums.md) | Small fingerprint to detect corrupted data | 💤 Low |

> [!note] Want more depth?
> The full-stack networking notes in [Computer Networks](../../computer-networks/README.md) go deeper into OSI, subnetting, HTTP headers and TLS.
