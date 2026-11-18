# Q&A · Availability

[← Back to System Design](../README.md) · [Read the notes →](../03-availability.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

> [!question]- 🟢 Q1. What is availability?
> The percentage of time a system is up and successfully serving requests. Best measured from the user's view: successful requests divided by total requests. We usually express it in "nines", like 99.9%.

> [!question]- 🟢 Q2. How much downtime does 99.9% vs 99.99% allow?
> 99.9% is about 8.8 hours a year, or ~43 minutes a month. 99.99% is about 52 minutes a year, or ~4 minutes a month. Each extra nine means 10x less downtime and a lot more cost.

> [!question]- 🟢 Q3. What is a single point of failure?
> Any component that brings the whole system down when it fails — one database, one load balancer, one data center. You remove it with redundancy: a second copy plus automatic failover.

> [!question]- 🟢 Q4. Availability vs reliability vs durability?
> Available means the system answers. Reliable means it answers correctly, even when parts fail. Durable means saved data is never lost. A system can be up but return wrong data, or up but have lost yesterday's writes.

> [!question]- 🟡 Q5. Explain SLI, SLO and SLA.
> SLI is the metric, like % of requests that succeed under 500 ms. SLO is the internal target, like 99.9% over 30 days. SLA is the contract with customers that has a penalty, and it's set looser than the SLO so we get a warning before we owe money.

> [!question]- 🟡 Q6. What is an error budget?
> The amount of failure the SLO allows — with a 99.9% SLO, about 43 minutes a month. If budget is left, we ship fast; if it's used up, we freeze risky releases and work on stability. It balances speed against reliability.

> [!question]- 🟡 Q7. What are MTBF and MTTR, and which would you focus on?
> MTBF is the average time between failures, MTTR the average time to recover. Availability = MTBF / (MTBF + MTTR). I focus on MTTR — failures will happen anyway, but automatic failover and good alerting can cut recovery from hours to seconds.

> [!question]- 🟡 Q8. Three services each at 99.9% are in the request path. What's the total availability?
> In series they multiply: 0.999³ ≈ 99.7%. Every extra dependency lowers availability. If instead I run two copies of a 99.9% service in parallel, it fails only when both fail: 1 − 0.001² = 99.9999%.

> [!question]- 🟡 Q9. Active-active vs active-passive?
> Active-passive: one node serves traffic, a standby takes over on failure — simpler, but standby capacity sits idle and failover takes seconds to minutes. Active-active: all nodes serve traffic, failover is near instant, but syncing writes between nodes is harder. I use active-active for stateless app servers and active-passive for a single-writer database.

> [!question]- 🟡 Q10. What are RPO and RTO?
> RPO is how much data we can afford to lose — sync replication gives RPO 0, async gives a few seconds. RTO is how long we can be down — auto failover is ~1 minute, restoring from backup is hours. Tighter numbers cost more, so I pick them per feature.

> [!question]- 🟡 Q11. How does a circuit breaker improve availability?
> It watches calls to a dependency. After too many failures it "opens" and fails fast for a while, instead of letting every request hang on timeouts. That frees threads, gives the dependency time to recover, and lets us return a fallback.

> [!question]- 🔴 Q12. What is split brain and how do you prevent it?
> A network partition makes the standby think the primary is dead, so it promotes itself while the old primary is still accepting writes — now two primaries, and data diverges. Prevent it with a quorum (majority of 3+ nodes elects the leader) and fencing (cut off the old primary before promoting the new one).

> [!question]- 🔴 Q13. We have 3 app servers at 80% CPU. One crashes and soon all three are down. Why?
> Cascading failure: the dead server's load moves to the other two, pushing them to ~120% of their capacity, so they crash too. Fix: keep N+1 headroom (each node under ~60%), auto-scale, and use load shedding so overloaded servers reject extra requests fast instead of dying.

> [!question]- 🔴 Q14. The recommendations service is slow and now the whole app is down. How would you have prevented this?
> One non-critical dependency used up all threads. I'd add timeouts on the call, a circuit breaker, a fallback like "trending posts", and a bulkhead — a separate connection pool — so recommendations can never starve the core feed. Retries only with backoff and jitter, to avoid a retry storm.

> [!question]- 🔴 Q15. Design for 99.99% availability. What do you put in place?
> No single points of failure: stateless app servers across multiple AZs behind a redundant load balancer, a database with a sync or semi-sync standby in another AZ and automatic failover, replicated cache. Then timeouts, circuit breakers and graceful degradation, canary deploys with auto-rollback, and alerts on user-facing error rates so MTTR is minutes. 4 minutes a month leaves no room for manual recovery — everything must be automatic.

> [!question]- 🔴 Q16. Would you choose availability or consistency?
> It depends on the feature. During a network split, likes, feeds and view counts should stay available even if slightly stale. Payments and balances should stay consistent — better to show "try again" than double-charge. That's the CAP trade-off, and I decide it per feature, not for the whole app.
