# IP Addressing & Subnetting

> **Subject:** Computer Networks  
> **Status:** ✅ done  
> **Last reviewed:** 2026-10-29

[← Back to Computer Networks](README.md)

## Overview

An **IP address** is a logical address at the **Network layer (L3)**. It identifies a device's interface on a network and lets routers forward packets across networks. A MAC address gets a frame to the next hop. An IP address gets a packet to its final destination.

There are two versions:

- **IPv4**: 32-bit, e.g. `192.168.1.10`, about 4.3 billion addresses (exhausted).
- **IPv6**: 128-bit, e.g. `2001:db8::1`, about 3.4 × 10³⁸ addresses.

---

## IPv4 structure

32 bits are split into **4 octets** (8 bits each) and written in **dotted decimal**. Each octet ranges from 0 to 255.

```
192      .168      .1        .10
11000000  10101000  00000001  00001010
```

### Binary ↔ decimal (memorise this row)

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|---|---|---|---|---|---|---|---|---|
| Value | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |

Example: `200` = 128 + 64 + 8 = `11001000`

Common mask octets: `128, 192, 224, 240, 248, 252, 254, 255` (1, 2, 3 … 8 leading ones).

### Network part vs host part

Every IPv4 address has two parts:

- **Network ID**: which network the address belongs to. Routers only look at this part.
- **Host ID**: which device on that network.

The **subnet mask** says where the split is. A `1` bit in the mask means network, and a `0` bit means host.

```
IP:    192.168.1.10    11000000.10101000.00000001.00001010
Mask:  255.255.255.0   11111111.11111111.11111111.00000000
                       |------- network (24) -------|host(8)|
```

---

## Classful addressing (historical, still asked in interviews)

Before 1993, the first bits of the address fixed its class.

| Class | Leading bits | First-octet range | Default mask | Networks | Usable hosts / network | Use |
|---|---|---|---|---|---|---|
| A | `0` | 1 – 126 | /8 (255.0.0.0) | 126 | 16,777,214 | Very large orgs |
| B | `10` | 128 – 191 | /16 (255.255.0.0) | 16,384 | 65,534 | Medium orgs |
| C | `110` | 192 – 223 | /24 (255.255.255.0) | 2,097,152 | 254 | Small networks |
| D | `1110` | 224 – 239 | — | — | — | Multicast |
| E | `1111` | 240 – 255 | — | — | — | Reserved / experimental |

- `127.x.x.x` is technically in class A's range, but it is reserved for **loopback**.
- `0.x.x.x` is reserved for "this network".

**Why classful addressing died:** it wasted addresses. An org needing 300 hosts was too big for a class C (254 hosts), so it got a class B and left about 65,000 addresses unused. **CIDR** replaced it.

---

## CIDR (Classless Inter-Domain Routing)

CIDR lets the network/host split fall on **any bit**. It is written as `address/prefix-length`, for example `192.168.1.0/24`, where 24 is the number of network bits.

### Formulas

- Host bits: **h = 32 − prefix**
- Total addresses: **2ʰ**
- Usable hosts: **2ʰ − 2**. The first address is the **network address** and the last is the **broadcast address**.
- Number of subnets when you borrow `n` bits: **2ⁿ**

### CIDR cheat sheet

| Prefix | Subnet mask | Total addresses | Usable hosts |
|---|---|---|---|
| /8 | 255.0.0.0 | 16,777,216 | 16,777,214 |
| /16 | 255.255.0.0 | 65,536 | 65,534 |
| /20 | 255.255.240.0 | 4,096 | 4,094 |
| /22 | 255.255.252.0 | 1,024 | 1,022 |
| /23 | 255.255.254.0 | 512 | 510 |
| /24 | 255.255.255.0 | 256 | 254 |
| /25 | 255.255.255.128 | 128 | 126 |
| /26 | 255.255.255.192 | 64 | 62 |
| /27 | 255.255.255.224 | 32 | 30 |
| /28 | 255.255.255.240 | 16 | 14 |
| /29 | 255.255.255.248 | 8 | 6 |
| /30 | 255.255.255.252 | 4 | 2 (classic point-to-point link) |
| /31 | 255.255.255.254 | 2 | 2 (point-to-point, RFC 3021; no network/broadcast) |
| /32 | 255.255.255.255 | 1 | 1 (single host route) |

> **Tip:** each step down in prefix doubles the size: /24 = 256, /23 = 512, /22 = 1024 …

---

## Subnetting: how to solve any question

### Method 1: bitwise AND (the "why")

**Network address = IP AND mask.**

```
IP:      192.168.1.130  → 11000000.10101000.00000001.10000010
Mask:    /26            → 11111111.11111111.11111111.11000000
AND:     192.168.1.128  → 11000000.10101000.00000001.10000000
```

### Method 2: block size / "magic number" (the fast way)

1. Find the **interesting octet**: the one where the mask is neither 255 nor 0.
2. **Block size = 256 − mask value in that octet.**
3. Subnets start at multiples of the block size. Find the multiple just at or below the IP's value in that octet.
4. Broadcast = next subnet start − 1.

### Worked example 1: find network, broadcast and host range

**Q:** `172.16.45.130/20`

- /20 means the mask is `255.255.240.0`. The interesting octet is the 3rd. Block size = 256 − 240 = **16**.
- Multiples of 16 are 0, 16, 32, **48** … The value 45 falls in the **32–47** block.

| | Value |
|---|---|
| Network address | `172.16.32.0` |
| First usable | `172.16.32.1` |
| Last usable | `172.16.47.254` |
| Broadcast | `172.16.47.255` |
| Usable hosts | 2¹² − 2 = **4,094** |

### Worked example 2: split a network into equal subnets

**Q:** Split `192.168.10.0/24` into 4 subnets.

- 4 subnets need 2ⁿ ≥ 4, so n = 2 borrowed bits. The new prefix is **/26**, with block size 64 and 62 hosts each.

| Subnet | Network | Usable range | Broadcast |
|---|---|---|---|
| 1 | 192.168.10.0/26 | .1 – .62 | .63 |
| 2 | 192.168.10.64/26 | .65 – .126 | .127 |
| 3 | 192.168.10.128/26 | .129 – .190 | .191 |
| 4 | 192.168.10.192/26 | .193 – .254 | .255 |

### Worked example 3: pick a mask for a host count

**Q:** What is the smallest subnet for 50 hosts?

- You need 2ʰ − 2 ≥ 50, so h = 6 (62 hosts). The prefix is **/26** (`255.255.255.192`).

### Are two hosts on the same subnet?

AND each IP with the mask. If both results match, the hosts are on the same subnet and talk directly (using ARP for the MAC). Otherwise the packet goes to the **default gateway**.

`10.0.5.20/23` and `10.0.4.200/23`: block size in the 3rd octet is 2, so both are in `10.0.4.0/23` and are on the **same subnet**.

---

## VLSM (Variable Length Subnet Masking)

VLSM gives different subnets **different mask sizes** so fewer addresses go to waste. **Always allocate the largest requirement first.**

**Q:** From `192.168.1.0/24`, create subnets for 100, 50, 20 and 2 hosts.

| Need | Prefix (hosts) | Network | Usable range | Broadcast |
|---|---|---|---|---|
| 100 | /25 (126) | 192.168.1.0/25 | .1 – .126 | .127 |
| 50 | /26 (62) | 192.168.1.128/26 | .129 – .190 | .191 |
| 20 | /27 (30) | 192.168.1.192/27 | .193 – .222 | .223 |
| 2 | /30 (2) | 192.168.1.224/30 | .225 – .226 | .227 |

`192.168.1.228` – `.255` is left free for future use.

## Supernetting / route aggregation

Supernetting is the opposite of subnetting: it combines contiguous networks into one route, which keeps routing tables small.

`192.168.0.0/24` + `192.168.1.0/24` + `192.168.2.0/24` + `192.168.3.0/24` → **`192.168.0.0/22`**

The first 22 bits are common to all four networks (3rd octet 0–3 = `000000xx`).

---

## Public vs private addresses

**Private ranges (RFC 1918)** are not routable on the internet. You can reuse them inside any network.

| Range | CIDR | Size | Typical use |
|---|---|---|---|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | 16.7M | Enterprises, cloud VPCs |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | 1M | Docker default (172.17.0.0/16), AWS default VPC |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | 65K | Home routers |

> **Trap:** `172.32.0.1` is **public**. Only 172.**16–31** is private.

## Special / reserved addresses

| Address / range | Meaning |
|---|---|
| `0.0.0.0` | "This host / any address". A server bound to `0.0.0.0` listens on all interfaces. In routing, `0.0.0.0/0` is the **default route**. |
| `127.0.0.0/8` (usually `127.0.0.1`) | **Loopback** (`localhost`). Traffic never leaves the machine. |
| `169.254.0.0/16` | **Link-local / APIPA**. The OS assigns one when DHCP fails, so seeing it means "DHCP isn't working". |
| `255.255.255.255` | **Limited broadcast** to everyone on the local network. Routers never forward it. |
| Last address of a subnet | **Directed broadcast** for that subnet |
| `100.64.0.0/10` | **Carrier-grade NAT** (ISP shared space, RFC 6598) |
| `224.0.0.0/4` | **Multicast** (class D) |
| `240.0.0.0/4` | Reserved (class E) |
| `192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24` | Documentation / examples only |

## Types of communication

| Type | Delivery | Example |
|---|---|---|
| **Unicast** | One → one | Normal web request |
| **Broadcast** | One → all on the local network | ARP request, DHCP Discover (IPv4 only) |
| **Multicast** | One → a subscribed group | IPTV, OSPF (`224.0.0.5`) |
| **Anycast** | One → **nearest** of many nodes sharing an IP | DNS root servers, Cloudflare `1.1.1.1`, CDNs |

---

## NAT (Network Address Translation)

NAT lets many private devices share one public IP. The router rewrites the source IP (and port) on the way out and reverses the change on replies, using a **translation table**.

```
Laptop 192.168.1.10:51000 ─┐                          ┌─> 93.184.216.34:443
Phone  192.168.1.11:51000 ─┼─ Router (public 203.0.113.5) ─┤
                           │  203.0.113.5:40001 ↔ 192.168.1.10:51000
                           │  203.0.113.5:40002 ↔ 192.168.1.11:51000
```

| Type | What it does |
|---|---|
| **Static NAT** | 1 private ↔ 1 public (fixed mapping) |
| **Dynamic NAT** | Private IPs take public IPs from a pool |
| **PAT / NAT overload** | Many private → 1 public, told apart by **port**. This is what home routers do. |
| **SNAT / DNAT** | Rewrites the **source** (outbound) vs the **destination** (port forwarding, load balancers) |

- **Pros:** saves IPv4 addresses and hides the internal layout.
- **Cons:** breaks end-to-end connectivity, so inbound connections need port forwarding. It also complicates P2P, VoIP and gaming (which need NAT traversal via STUN/TURN), and adds per-connection state at the router.
- **NAT is not a firewall**, even though it incidentally blocks unsolicited inbound traffic.

---

## How a device gets an IP

### Static vs dynamic

- **Static**: configured by hand and never changes. Used for servers, routers and printers.
- **Dynamic**: leased from a **DHCP** server for a limited time.

### DHCP: the DORA process (UDP, server 67 / client 68)

```
Client                                  DHCP Server
  | -- DISCOVER (broadcast) -----------> |   "Any DHCP servers?"
  | <------------------------ OFFER ---- |   "You can have 192.168.1.10"
  | -- REQUEST (broadcast) ------------> |   "I'll take 192.168.1.10"
  | <------------------------- ACK ----- |   "Confirmed, lease = 24h"
```

DHCP hands out the **IP, subnet mask, default gateway, DNS servers and lease time**. The client tries to renew at 50% of the lease (T1). If DHCP is on another subnet, a **DHCP relay** (`ip helper-address`) forwards the broadcasts.

### ARP: IP → MAC on the local network

1. Host A wants to reach `192.168.1.20` on the same subnet but doesn't know its MAC address.
2. A broadcasts: *"Who has 192.168.1.20? Tell 192.168.1.10."*
3. The owner replies by **unicast** with its MAC. A caches the mapping (`arp -a`).
4. For a destination on a **different subnet**, A ARPs for the **gateway's** MAC instead. The destination IP stays the same, but the frame's destination MAC is the router's.

- **Gratuitous ARP**: a host announces its own IP→MAC mapping. It is used to detect duplicate IPs and for failover.
- **ARP spoofing**: an attacker sends fake ARP replies to become a man-in-the-middle. ARP has no authentication.

> **Key interview point:** across a path, the **IP addresses stay the same end to end** (unless NAT rewrites them), while the **MAC addresses change at every hop**.

---

## IPv4 header (key fields)

The header is 20 bytes minimum and 60 maximum (with options).

| Field | Purpose |
|---|---|
| Version | 4 |
| IHL | Header length |
| Total Length | Header + data, max 65,535 bytes |
| Identification, Flags (DF/MF), Fragment Offset | Fragmentation and reassembly |
| **TTL** | Decremented at each router. At 0 the packet is dropped and an ICMP "Time Exceeded" is sent, which is how `traceroute` works. It prevents routing loops. |
| Protocol | Upper-layer protocol: 1 = ICMP, 6 = TCP, 17 = UDP |
| Header Checksum | Header only. Recomputed at each hop because TTL changes. |
| Source / Destination IP | 32 bits each |

---

## IPv6

### Why IPv6

IPv4 ran out (IANA's free pool was exhausted in 2011). NAT is a workaround, not a fix. IPv6 provides 128-bit addresses, so there's no need for NAT.

### Format

8 groups of 4 hex digits (16 bits each), separated by `:`.

```
Full:        2001:0db8:0000:0000:0000:ff00:0042:8329
Compressed:  2001:db8::ff00:42:8329
```

**Compression rules:**

1. You may drop the **leading zeros** in each group (`0db8` → `db8`, `0042` → `42`).
2. You may replace **one** run of consecutive all-zero groups with `::`. You can only do this **once**; otherwise the address would be ambiguous.

### Address types

| Type | Prefix | Like IPv4's… |
|---|---|---|
| Global unicast | `2000::/3` | Public IP |
| Unique local | `fc00::/7` (in practice `fd00::/8`) | Private (RFC 1918) |
| Link-local | `fe80::/10` | 169.254.x.x. **Every** IPv6 interface always has one. |
| Loopback | `::1` | 127.0.0.1 |
| Unspecified | `::` | 0.0.0.0 |
| Multicast | `ff00::/8` | 224.0.0.0/4 |

- **IPv6 has no broadcast.** Multicast and anycast replace it.
- A standard LAN subnet is **/64**: the first 64 bits are the network and the last 64 are the interface ID. Sites usually get a /48 or /56.
- **SLAAC** (Stateless Address Autoconfiguration) lets a host build its own address from the router's prefix, with no DHCP needed. DHCPv6 also exists.
- **NDP** (Neighbor Discovery Protocol, built on ICMPv6) replaces **ARP**.

### IPv4 vs IPv6

| | IPv4 | IPv6 |
|---|---|---|
| Size | 32-bit | 128-bit |
| Notation | Dotted decimal | Hex, colon-separated |
| Address count | ~4.3 × 10⁹ | ~3.4 × 10³⁸ |
| Header | 20–60 bytes, variable | 40 bytes fixed (extension headers chain) |
| Header checksum | Yes | No (left to L2/L4, which makes routing faster) |
| Fragmentation | Routers and sender | **Sender only** (Path MTU Discovery) |
| Broadcast | Yes | No (multicast instead) |
| Address config | Manual / DHCP | SLAAC / DHCPv6 / manual |
| IP → MAC | ARP | NDP |
| NAT | Common | Not needed |
| IPsec | Optional | Built into the spec (originally mandatory) |

### Transition mechanisms

- **Dual stack**: devices run IPv4 and IPv6 side by side. This is the most common approach.
- **Tunneling**: IPv6 packets are carried inside IPv4 (6to4, Teredo).
- **NAT64 / DNS64**: lets IPv6-only clients reach IPv4-only servers.

---

## Useful commands

| Task | Linux | macOS | Windows |
|---|---|---|---|
| Show IPs | `ip addr` | `ifconfig` / `ipconfig getifaddr en0` | `ipconfig` |
| Routing table / gateway | `ip route` | `netstat -rn` | `route print` |
| ARP cache | `ip neigh` | `arp -a` | `arp -a` |
| Reachability | `ping` | `ping` | `ping` |
| Path / hops | `traceroute` | `traceroute` | `tracert` |
| Public IP | `curl ifconfig.me` | `curl ifconfig.me` | — |
| Subnet calc | `ipcalc 10.0.0.0/20` | `brew install ipcalc` | — |

---

## Common traps

- **Usable hosts = 2ʰ − 2**, not 2ʰ (the exceptions are /31 and /32).
- `172.32.x.x` is **public**. The private range is only 172.16–172.31.
- `127.x` is not class A in practice. It is loopback.
- **IP vs MAC:** IP is logical and changes across networks. MAC is physical and burned into the NIC. MAC is used hop to hop, IP end to end.
- `::` can appear **only once** in an IPv6 address.
- A `169.254.x.x` address means **DHCP failed**.
- With VLSM, allocate the **largest** subnet first, or the blocks won't align.

## Quick revision

- IPv4 = 32 bits / 4 octets. IPv6 = 128 bits / 8 hex groups.
- Mask/prefix splits the network and host parts. Network = IP AND mask.
- Hosts = 2^(32−prefix) − 2. Block size = 256 − interesting mask octet.
- Private: 10/8, 172.16/12, 192.168/16. Loopback 127/8. APIPA 169.254/16.
- CIDR replaced classful addressing. VLSM uses varied masks. Supernetting aggregates routes.
- NAT/PAT: many private IPs share one public IP via ports.
- DHCP = DORA (UDP 67/68). ARP = IP→MAC (broadcast request, unicast reply).
- IPv6: no broadcast, no NAT needed, NDP replaces ARP, SLAAC, /64 subnets, 40-byte fixed header.

## Interview / self-test questions

1. What's the difference between an IP address and a MAC address?
   <details><summary>Answer</summary>

   An IP address is a **logical** L3 address. It is assigned by configuration or DHCP, changes when a device moves networks, and is used for **end-to-end** routing. A MAC address is a 48-bit **physical** L2 address burned into the NIC, used for delivery **within one link / hop to hop**. Along a path the source and destination IPs stay the same (unless NAT rewrites them), while the MACs are rewritten at every router.

   </details>

2. How many usable hosts are in a /27? What's its mask?
   <details><summary>Answer</summary>

   h = 32 − 27 = 5, so 2⁵ − 2 = **30 hosts**. The mask is `255.255.255.224`.

   </details>

3. Find the network, broadcast and usable range for `10.1.1.77/28`.
   <details><summary>Answer</summary>

   The block size is 256 − 240 = 16. Multiples are 64 and 80, so 77 falls in the **64–79** block.
   Network `10.1.1.64`, usable `10.1.1.65 – 10.1.1.78`, broadcast `10.1.1.79`.

   </details>

4. Why do we subtract 2 when counting hosts?
   <details><summary>Answer</summary>

   The all-zeros host address is the **network address** (it identifies the subnet itself) and the all-ones host address is the **directed broadcast**. Neither can be assigned to a host. The exception is /31 point-to-point links (RFC 3021), which use both addresses.

   </details>

5. What is CIDR and why was it introduced?
   <details><summary>Answer</summary>

   Classless Inter-Domain Routing (1993) lets the network/host boundary fall at any bit, written as `/prefix`. It replaced fixed class A/B/C boundaries that wasted huge numbers of addresses. It also allows **route aggregation** (supernetting), which keeps internet routing tables small.

   </details>

6. What happens when your laptop gets `169.254.x.x`?
   <details><summary>Answer</summary>

   It couldn't reach a DHCP server, so the OS self-assigned an **APIPA / link-local** address. The laptop can talk only to other link-local hosts on the same segment, with no gateway and no internet.

   </details>

7. How does a home network with one public IP let 10 devices browse at once?
   <details><summary>Answer</summary>

   With **PAT (NAT overload)**. The router rewrites each outgoing packet's private source IP:port to its public IP plus a unique port, and records the mapping in a NAT table. Replies come back to that public port, and the router translates them back to the right private device.

   </details>

8. Explain DHCP's DORA process.
   <details><summary>Answer</summary>

   1. **Discover**: the client broadcasts to find servers.
   2. **Offer**: a server offers an IP plus config.
   3. **Request**: the client broadcasts that it accepts that offer, which also tells any other servers their offers were declined.
   4. **Acknowledge**: the server confirms the lease.

   DHCP runs over UDP, with the server on port 67 and the client on port 68.

   </details>

9. Host A (`192.168.1.10/24`) sends to `8.8.8.8`. Walk through what happens at L2/L3.
   <details><summary>Answer</summary>

   1. A ANDs the destination with its mask, finds that `8.8.8.8` is on a different network, and decides to send to the **default gateway**.
   2. A ARPs for the gateway's MAC (unless it is cached).
   3. A builds a frame with dst MAC = gateway, src IP = `192.168.1.10`, dst IP = `8.8.8.8`.
   4. The router applies NAT (src IP becomes its public IP), decrements TTL, and forwards with new MACs for the next hop.
   5. Each router hop repeats the forwarding: the MACs change and the IPs stay the same.

   </details>

10. Summarise `10.0.0.0/24`, `10.0.1.0/24`, `10.0.2.0/24` and `10.0.3.0/24` into one route.
    <details><summary>Answer</summary>

    The 3rd octets 0–3 share their first 6 bits, so 16 + 6 = 22 bits are common: **`10.0.0.0/22`**.

    </details>

11. Why does IPv6 not need NAT, and what replaced ARP and broadcast?
    <details><summary>Answer</summary>

    With 2¹²⁸ addresses, every device can have a globally unique address, so there's no need to share one. **NDP** (ICMPv6) replaces ARP. **Multicast** (e.g. solicited-node multicast) replaces broadcast.

    </details>

12. Compress `fe80:0000:0000:0000:0202:b3ff:fe1e:8329`.
    <details><summary>Answer</summary>

    `fe80::202:b3ff:fe1e:8329`

    </details>

13. Is `172.20.5.1` public or private? What about `172.40.5.1`?
    <details><summary>Answer</summary>

    `172.20.5.1` is **private** (inside 172.16.0.0/12, i.e. 172.16–172.31). `172.40.5.1` is **public**.

    </details>

14. What does binding a server to `0.0.0.0` vs `127.0.0.1` mean?
    <details><summary>Answer</summary>

    `127.0.0.1` accepts connections only from the same machine. `0.0.0.0` listens on **all interfaces**, so other machines can connect. This is a common Docker/dev-server gotcha: a server bound to `127.0.0.1` inside a container can't be reached through a published port.

    </details>

15. What is anycast? Give a real example.
    <details><summary>Answer</summary>

    Many servers in different places announce the **same IP** via BGP, and routing delivers each packet to the topologically **nearest** one. Examples are DNS root servers, Cloudflare `1.1.1.1`, Google `8.8.8.8` and CDN edges. It gives low latency and DDoS absorption.

    </details>

## References

- *Computer Networking: A Top-Down Approach* (Kurose & Ross), Ch. 4 (Network Layer)
- [RFC 1918: Private address space](https://www.rfc-editor.org/rfc/rfc1918)
- [RFC 4632: CIDR](https://www.rfc-editor.org/rfc/rfc4632)
- [RFC 4291: IPv6 addressing architecture](https://www.rfc-editor.org/rfc/rfc4291)
- [Cloudflare: What is my IP address?](https://www.cloudflare.com/learning/dns/glossary/what-is-my-ip-address/)
- [subnet-calculator.com](https://www.subnet-calculator.com/) for practising
