# Core Concepts · Practice Questions

[← Back to Core Concepts](../README.md) · [Q&A with answers →](README.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Open the topic's Q&A file (linked under each heading) to check.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**118 questions** across 7 topics.

---

## 01 · Scalability

[Answers →](01-scalability-qa.md) · 15 questions

- [ ] 🟢 Q1. What is scalability?
- [ ] 🟢 Q2. Vertical vs horizontal scaling?
- [ ] 🟢 Q3. Performance vs scalability?
- [ ] 🟢 Q4. What is a stateless server and why does it matter?
- [ ] 🟡 Q5. What are sticky sessions and why avoid them?
- [ ] 🟡 Q6. Scalability vs elasticity?
- [ ] 🟡 Q7. How do you scale a database?
- [ ] 🟡 Q8. What's the downside of read replicas?
- [ ] 🟡 Q9. What is sharding and its problems?
- [ ] 🟡 Q10. How do message queues help scalability?
- [ ] 🟡 Q11. What is the scale cube?
- [ ] 🔴 Q12. You added more app servers but the site is still slow. Why?
- [ ] 🔴 Q13. How would you scale an app from 1 user to 10 million?
- [ ] 🔴 Q14. A flash sale is at 12:00 and traffic will be 20x. What do you do?
- [ ] 🔴 Q15. Why use p99 latency instead of average?

---

## 02 · Availability

[Answers →](02-availability-qa.md) · 21 questions

- [ ] 🟢 Q1. What is availability?
- [ ] 🟢 Q2. How much downtime does 99.9% vs 99.99% allow?
- [ ] 🟢 Q3. What is a single point of failure?
- [ ] 🟢 Q4. Availability vs reliability vs durability?
- [ ] 🟡 Q5. Explain SLI, SLO and SLA.
- [ ] 🟡 Q6. What is an error budget?
- [ ] 🟡 Q7. What are MTBF and MTTR, and which would you focus on?
- [ ] 🟡 Q8. Three services each at 99.9% are in the request path. What's the total availability?
- [ ] 🟡 Q9. Active-active vs active-passive?
- [ ] 🟡 Q10. What are RPO and RTO?
- [ ] 🟡 Q11. How does a circuit breaker improve availability?
- [ ] 🟡 Q12. What is a cascading failure?
- [ ] 🟡 Q13. Why is a slow service often worse than a dead one?
- [ ] 🔴 Q14. What is split brain and how do you prevent it?
- [ ] 🔴 Q15. We have 3 app servers at 80% CPU. One crashes and soon all three are down. Why?
- [ ] 🔴 Q16. The recommendations service is slow and now the whole app is down. How would you have prevented this?
- [ ] 🔴 Q17. Redis restarted and a minute later the database went down. What happened and how do you prevent it?
- [ ] 🔴 Q18. How do retries cause outages, and how do you retry safely?
- [ ] 🔴 Q19. A crashed server keeps restarting and crashing again. Why?
- [ ] 🔴 Q20. Design for 99.99% availability. What do you put in place?
- [ ] 🔴 Q21. Would you choose availability or consistency?

---

## 03 · Reliability

[Answers →](03-reliability-qa.md) · 16 questions

- [ ] 🟢 Q1. What is reliability?
- [ ] 🟢 Q2. Reliability vs availability?
- [ ] 🟢 Q3. What's the difference between a fault and a failure?
- [ ] 🟢 Q4. What kinds of faults should you design for?
- [ ] 🟢 Q5. What does idempotent mean? Give examples.
- [ ] 🟡 Q6. Why are retries dangerous, and how do idempotency keys fix that?
- [ ] 🟡 Q7. Explain at-most-once, at-least-once, and exactly-once delivery.
- [ ] 🟡 Q8. What is the dual-write problem and the outbox pattern?
- [ ] 🟡 Q9. Is replication enough to protect your data?
- [ ] 🟡 Q10. How do you keep data correct when the app might have bugs?
- [ ] 🟡 Q11. How do you reduce human errors in production?
- [ ] 🟡 Q12. What is a gray failure?
- [ ] 🔴 Q13. Users report their post appears three times after one tap. What's happening and how do you fix it?
- [ ] 🔴 Q14. Some users were charged twice for a subscription. How would you design payments to prevent and detect this?
- [ ] 🔴 Q15. A post's like count shows 120 but only 80 people are in the likes list. Why, and how would you fix it?
- [ ] 🔴 Q16. Our notification service sometimes sends the same push 5 times and sometimes not at all. What would you change?

---

## 04 · Single Point of Failure

[Answers →](04-single-point-of-failure-qa.md) · 16 questions

- [ ] 🟢 Q1. What is a single point of failure?
- [ ] 🟢 Q2. Is every single component a SPOF?
- [ ] 🟢 Q3. How do you find SPOFs in a design?
- [ ] 🟢 Q4. What's the most common way to remove a SPOF?
- [ ] 🟢 Q5. What do N+1 and N+2 mean?
- [ ] 🟡 Q6. The load balancer protects app servers — but what protects the load balancer?
- [ ] 🟡 Q7. Why do coordination clusters like ZooKeeper or etcd use 3 or 5 nodes and not 2 or 4?
- [ ] 🟡 Q8. Name some hidden SPOFs that people often miss.
- [ ] 🟡 Q9. What is a correlated failure and why does it matter for SPOFs?
- [ ] 🟡 Q10. How do you handle a third-party dependency like a payment provider being a SPOF?
- [ ] 🟡 Q11. Should you always remove every SPOF?
- [ ] 🟡 Q12. How do you test that your system really has no SPOFs?
- [ ] 🔴 Q13. We have 8 app servers across 2 zones, but login and payments went down for 50 minutes while the feed still worked. What could it be?
- [ ] 🔴 Q14. At midnight every server started failing at the same moment, even though nothing was deployed. What do you suspect?
- [ ] 🔴 Q15. A bad config push took down all 10 regions at once. How would you redesign to limit this?
- [ ] 🔴 Q16. You're asked "where are the SPOFs?" about your design. How do you answer?

---

## 05 · Latency vs Throughput vs Bandwidth

[Answers →](05-latency-throughput-bandwidth-qa.md) · 17 questions

- [ ] 🟢 Q1. What's the difference between latency, throughput, and bandwidth?
- [ ] 🟢 Q2. Can a link have high bandwidth and high latency at the same time?
- [ ] 🟢 Q3. Why is throughput always lower than bandwidth?
- [ ] 🟢 Q4. My internet is 100 Mbps. How long does a 1 GB file take to download?
- [ ] 🟢 Q5. What's the difference between latency, processing time, and response time?
- [ ] 🟢 Q6. Why use percentiles like p99 instead of average latency?
- [ ] 🟡 Q7. Where does latency come from on a network request?
- [ ] 🟡 Q8. Give some rough latency numbers you keep in your head.
- [ ] 🟡 Q9. What is Little's Law and how do you use it?
- [ ] 🟡 Q10. Why does latency shoot up as utilization approaches 100%?
- [ ] 🟡 Q11. How do you improve latency vs throughput vs bandwidth?
- [ ] 🟡 Q12. Give an example where improving throughput hurts latency.
- [ ] 🔴 Q13. What is tail amplification, and how do you deal with it?
- [ ] 🔴 Q14. Users in India say the feed takes 3 seconds, but US users say it's instant. Server time is 50 ms. What's going on?
- [ ] 🔴 Q15. Our dashboard shows average latency of 120 ms, but users keep complaining the app is slow. How do you investigate?
- [ ] 🔴 Q16. Every evening at 9 PM the app gets slow, but nothing is failing and CPU is at 92%. What's happening and what do you do?
- [ ] 🔴 Q17. Video uploads take 15 minutes for a 1 GB file. How would you make them faster?

---

## 06 · Consistent Hashing

[Answers →](06-consistent-hashing-qa.md) · 16 questions

- [ ] 🟢 Q1. What is consistent hashing, in one or two sentences?
- [ ] 🟢 Q2. What's wrong with `hash(key) % N`?
- [ ] 🟢 Q3. How does the hash ring decide which server owns a key?
- [ ] 🟢 Q4. What happens when you add or remove a server on the ring?
- [ ] 🟢 Q5. What are virtual nodes and why do we need them?
- [ ] 🟡 Q6. How do you handle servers with different capacities?
- [ ] 🟡 Q7. How is replication done on a consistent hash ring?
- [ ] 🟡 Q8. How would you implement the lookup? What's the complexity?
- [ ] 🟡 Q9. Does consistent hashing solve hot keys?
- [ ] 🟡 Q10. How do all clients agree on what the ring looks like?
- [ ] 🟡 Q11. Name real systems that use consistent hashing.
- [ ] 🟡 Q12. How is Redis Cluster's approach different from a hash ring?
- [ ] 🔴 Q13. We added a 5th Redis node and the database immediately went to 100% CPU. What happened and how do you fix it?
- [ ] 🔴 Q14. One cache node died, and then its neighbour died a few minutes later, and then the next one. Why?
- [ ] 🔴 Q15. A celebrity posts and one cache node hits 100% CPU while the others are idle. How do you handle it?
- [ ] 🔴 Q16. When would you choose rendezvous hashing or jump hash instead of a ring?

---

## 07 · CAP Theorem

[Answers →](07-cap-theorem-qa.md) · 17 questions

- [ ] 🟢 Q1. What is the CAP theorem?
- [ ] 🟢 Q2. What do C, A, and P mean exactly?
- [ ] 🟢 Q3. Is CAP's C the same as the C in ACID?
- [ ] 🟢 Q4. Why can't you just "pick any 2 of 3"?
- [ ] 🟢 Q5. Is a CA system possible?
- [ ] 🟡 Q6. Is CAP availability the same as 99.99% uptime?
- [ ] 🟡 Q7. Walk me through what a CP system does during a partition.
- [ ] 🟡 Q8. And what does an AP system do?
- [ ] 🟡 Q9. In a social media app, which features would you make CP and which AP?
- [ ] 🟡 Q10. Name some CP-leaning and AP-leaning databases.
- [ ] 🟡 Q11. What does R + W > N mean?
- [ ] 🟡 Q12. What consistency models sit between strong and eventual?
- [ ] 🔴 Q13. How do AP systems resolve conflicts, and what's the danger with last-write-wins?
- [ ] 🔴 Q14. What is PACELC and why does it matter?
- [ ] 🔴 Q15. Scenario: during a network split, two users in different regions both got the same username. What went wrong and how do you fix it?
- [ ] 🔴 Q16. Scenario: users complain their like count jumps 120 → 98 → 120 on refresh, and after editing their bio they still see the old one. How do you fix it without going fully CP?
- [ ] 🔴 Q17. How would you bring up CAP in a system design interview?
