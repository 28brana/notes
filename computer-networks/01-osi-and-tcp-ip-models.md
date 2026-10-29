# OSI and TCP/IP Models

> **Subject:** Computer Networks  
> **Status:** ✅ done  
> **Last reviewed:** 2026-10-29

[← Back to Computer Networks](README.md)

## Overview

Network models split communication into **layers**, each with one job, so each layer only talks to the one directly above and below it. The **OSI model** (7 layers) is the conceptual reference; the **TCP/IP model** (4 layers) is what the internet actually uses.

## The OSI model

| # | Layer | Job | Data unit | Examples |
|---|---|---|---|---|
| 7 | Application | Network services for apps | Data | HTTP, FTP, SMTP, DNS |
| 6 | Presentation | Encoding, encryption, compression | Data | TLS/SSL, JPEG, ASCII |
| 5 | Session | Open/manage/close sessions | Data | RPC, NetBIOS |
| 4 | Transport | End-to-end delivery, ports, reliability | Segment (TCP) / Datagram (UDP) | TCP, UDP |
| 3 | Network | Logical addressing & routing | Packet | IP, ICMP, routers |
| 2 | Data Link | Node-to-node delivery, MAC addressing | Frame | Ethernet, Wi-Fi, switches |
| 1 | Physical | Raw bits over the medium | Bits | Cables, fiber, radio, hubs |

**Mnemonic (7 → 1):** **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing  
**Mnemonic (1 → 7):** **P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way

## The TCP/IP model

| TCP/IP layer | Maps to OSI layers | Protocols |
|---|---|---|
| Application | 7, 6, 5 | HTTP, DNS, SMTP, SSH |
| Transport | 4 | TCP, UDP |
| Internet | 3 | IP, ICMP, ARP* |
| Network Access (Link) | 2, 1 | Ethernet, Wi-Fi |

\*ARP sits between layers 2 and 3.

## Encapsulation

Going **down** the stack, each layer wraps the data with its own header; going **up**, each layer strips it.

```
Application:  [            Data            ]
Transport:    [TCP hdr][      Data         ]                 → Segment
Network:      [IP hdr][TCP hdr][   Data    ]                 → Packet
Data Link:    [Eth hdr][IP hdr][TCP hdr][Data][Eth trailer]  → Frame
Physical:     0101101001...                                  → Bits
```

## Which device works at which layer?

| Device | Layer | Uses |
|---|---|---|
| Hub | 1 – Physical | Repeats bits to all ports |
| Switch | 2 – Data Link | MAC addresses |
| Router | 3 – Network | IP addresses |
| Load balancer | 4 or 7 | Ports (L4) or HTTP content (L7) |

## OSI vs TCP/IP

| | OSI | TCP/IP |
|---|---|---|
| Layers | 7 | 4 |
| Nature | Theoretical reference | Practical, implemented |
| Developed by | ISO | US DoD (ARPANET) |
| Use today | Teaching & troubleshooting vocabulary | Runs the internet |

## Quick revision

- OSI = 7 layers (reference); TCP/IP = 4 layers (real world).
- Transport = ports (TCP/UDP); Network = IP addresses; Data Link = MAC addresses.
- Data units: bits → frame → packet → segment → data.

## Self-test questions

1. At which layer does a router operate, and what address does it use?
   <details><summary>Answer</summary>

   Layer 3 (Network), using IP addresses.

   </details>

2. What is encapsulation?
   <details><summary>Answer</summary>

   Each layer adds its own header (and Data Link a trailer) to the data from the layer above as it moves down the stack; the receiver removes them going up.

   </details>

3. Where does TLS fit in the OSI model?
   <details><summary>Answer</summary>

   Usually placed at the Presentation layer (6) since it handles encryption; in TCP/IP it's part of the Application layer.

   </details>

## References

- *Computer Networking: A Top-Down Approach* — Kurose & Ross, Ch. 1
- [Cloudflare: What is the OSI model?](https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-model-osi/)
