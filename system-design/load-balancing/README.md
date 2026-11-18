# System Design · Load Balancing

[← Back to System Design](../README.md) · [Interview Q&A →](questions/load-balancing-qa.md) · [Practice questions 📝](questions/practice-questions.md)

How we spread traffic across many servers — inside one data center and across the world. Almost every design interview has a load balancer in the first diagram, so expect follow-ups on it.

| # | Topic | In one line | Priority |
|---|---|---|---|
| 01 | [Load Balancers](01-load-balancers.md) | A traffic cop in front of your servers — L4 vs L7, health checks, sticky sessions, avoiding the LB as a SPOF | 🔥 High |
| 02 | [Load Balancing Algorithms](02-load-balancing-algorithms.md) | How the LB picks a server — round robin, least connections, hashing, power of two choices | 🔥 High |
| 03 | [DNS Load Balancing](03-dns-load-balancing.md) | Spreading users across servers or regions by returning different IPs from DNS | ⚡ Medium |
| 04 | [Anycast Routing](04-anycast-routing.md) | Many locations share one IP, and the internet routes each user to the nearest one | 💤 Low |
