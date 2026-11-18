# Q&A · Single Point of Failure

[← Back to Core Concepts](../README.md) · [Read the notes →](../04-single-point-of-failure.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

> [!question]- 🟢 Q1. What is a single point of failure?
> Any one component whose failure takes down the whole system, or a critical feature of it. It has two parts: there's only one of it (or all copies fail together), and an important user action can't work without it. One database with no standby is the classic example.

> [!question]- 🟢 Q2. Is every single component a SPOF?
> No — only if it's on the critical path. A single analytics server dying doesn't stop users posting, so it's not a SPOF for the app. A single auth service is, because nobody can log in. It's about blast radius, not just the count.

> [!question]- 🟢 Q3. How do you find SPOFs in a design?
> I trace each important user flow end to end — read path, write path, background jobs, even the deploy path — and list every component and external dependency. For each I ask: how many copies are there, what breaks if it dies, and how fast do we recover? One copy plus a big impact means SPOF.

> [!question]- 🟢 Q4. What's the most common way to remove a SPOF?
> Redundancy: run more than one copy, spread across failure domains like racks and zones, with health checks and automatic failover. It's easiest for stateless components, so I move state out of app servers first — sessions to Redis, files to object storage.

> [!question]- 🟢 Q5. What do N+1 and N+2 mean?
> If peak load needs N servers, N+1 means one extra so you survive one failure at peak. N+2 means two extra, so you survive a failure while another server is down for patching or deploys. With peak needing 4 servers, N+1 is 5 and N+2 is 6.

> [!question]- 🟡 Q6. The load balancer protects app servers — but what protects the load balancer?
> Run two load balancers sharing a floating IP. They exchange heartbeats using VRRP, for example via keepalived, and if the active one dies the standby takes over the IP in 1–3 seconds. In the cloud, a managed load balancer already runs across multiple zones for you.

> [!question]- 🟡 Q7. Why do coordination clusters like ZooKeeper or etcd use 3 or 5 nodes and not 2 or 4?
> They need a majority — a quorum — to elect a leader and avoid split brain. With 2 nodes, losing one leaves 1 of 2, which isn't a majority, so 2 is no better than 1. 3 nodes tolerate 1 failure, 5 tolerate 2, and 4 tolerates the same as 3, so odd numbers give the best value.

> [!question]- 🟡 Q8. Name some hidden SPOFs that people often miss.
> TLS certificates or API keys that expire on all servers at once, a cron job pinned to one box, a single NAT gateway, leader nodes, long DNS TTLs that slow down failover, one cloud account holding prod and backups, a shared library every service uses, and the bus factor — one person who knows how to fix things.

> [!question]- 🟡 Q9. What is a correlated failure and why does it matter for SPOFs?
> It's when redundant copies fail together because they share something — the same rack, zone, software bug, deploy, certificate or dependency. Two copies that die together are really one copy, so the redundancy is fake. I fix it with anti-affinity across racks and zones, staggered canary rollouts, and asking "what single event could kill both?"

> [!question]- 🟡 Q10. How do you handle a third-party dependency like a payment provider being a SPOF?
> Options in order of cost: degrade gracefully so only checkout breaks, not the whole app; accept the order and queue the charge to retry later; or integrate a second provider behind the same interface. Users don't care whose outage it was, so the provider's SLA becomes part of my availability.

> [!question]- 🟡 Q11. Should you always remove every SPOF?
> No. I weigh likelihood times impact against the cost of the fix. A second NAT gateway costs a few dollars a month, so it's an easy yes. Multi-cloud costs a fortune for a very rare event, so most companies accept it. For accepted SPOFs I document the risk, keep backups and a tested runbook, and monitor closely to keep recovery time short.

> [!question]- 🟡 Q12. How do you test that your system really has no SPOFs?
> Chaos engineering and game days. Define steady state, form a hypothesis like "losing one Redis node won't hurt users", inject the failure, and watch metrics — starting small in staging and growing the blast radius. Game days also test people: have someone other than the usual expert run the database restore.

> [!question]- 🔴 Q13. We have 8 app servers across 2 zones, but login and payments went down for 50 minutes while the feed still worked. What could it be?
> Something shared by all servers on the path to external services broke — login and payments both call third parties, the feed doesn't. A likely cause is a single NAT gateway in one zone that all outbound traffic goes through, or a shared egress proxy or DNS resolver. I'd add one NAT gateway per zone, route each zone through its own, and add synthetic checks on outbound calls.

> [!question]- 🔴 Q14. At midnight every server started failing at the same moment, even though nothing was deployed. What do you suspect?
> Something time-based that's shared by every server: an expired TLS certificate or API key, a leap-second or date bug, or a scheduled job that ran everywhere at once. Redundancy doesn't help because every copy shares the cause. Fix: auto-renew certs with alerts 30 days before expiry, and treat time-based events as risks to test ahead of time.

> [!question]- 🔴 Q15. A bad config push took down all 10 regions at once. How would you redesign to limit this?
> The deploy pipeline itself was the SPOF, because it pushed to everything at once. I'd roll out config like code: validate in CI, canary to one cell or region, wait and compare metrics, then expand in waves with automatic rollback. A cell-based architecture also caps the blast radius — one bad change hurts 10% of users, not 100%.

> [!question]- 🔴 Q16. You're asked "where are the SPOFs?" about your design. How do you answer?
> I go box by box along the request path: DNS, CDN, load balancer, app, cache, database, queue, external APIs — saying how many copies each has, what breaks if it dies, and the fix with its cost. Then I mention hidden ones like certificates, NAT, leader nodes and the deploy pipeline. Finally I say which SPOFs I'd consciously accept, like a single cloud provider, and how we'd recover.
