# Distributed Systems · Practice Questions

[← Back to Distributed Systems](../README.md) · [Q&A with answers →](distributed-systems-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](distributed-systems-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**47 questions.**

---

## 01 · Challenges of Distribution

- [ ] 🟢 Q1. What makes distributed systems harder than a single machine?
- [ ] 🟡 Q2. Name some of the fallacies of distributed computing.
- [ ] 🟡 Q3. A request to another service timed out. What do you know about it?
- [ ] 🔴 Q4. You moved from 10 to 1,000 servers and now see "random" failures daily. Why, and how do you think about it?

---

## 02 · Network Partitions

- [ ] 🟢 Q5. What is a network partition?
- [ ] 🟡 Q6. Why do clusters use an odd number of nodes?
- [ ] 🔴 Q7. The link between your two data centers drops. DC-A has 3 DB nodes, DC-B has 2. What should happen?

---

## 03 · Split Brain

- [ ] 🟢 Q8. What is split brain?
- [ ] 🟡 Q9. How do you prevent split brain?
- [ ] 🔴 Q10. You have a primary and one standby with automatic failover, and twice this month you had two primaries. What's wrong?

---

## 04 · Heartbeats

- [ ] 🟢 Q11. What is a heartbeat and why is it used?
- [ ] 🟡 Q12. How do you choose the heartbeat timeout?
- [ ] 🔴 Q13. Your cluster keeps electing a new leader every few minutes even though nothing crashed. Why?

---

## 05 · Handling Failures

- [ ] 🟢 Q14. Why does every network call need a timeout?
- [ ] 🟡 Q15. What are exponential backoff and jitter, and why both?
- [ ] 🟡 Q16. What is a fencing token?
- [ ] 🔴 Q17. During an outage, your downstream service got 27 times its normal traffic. What happened?

---

## 06 · Consistency Models

- [ ] 🟢 Q18. What is eventual consistency?
- [ ] 🟡 Q19. What's the difference between linearizable and causal consistency?
- [ ] 🟡 Q20. What is read-your-writes consistency and how do you implement it?
- [ ] 🔴 Q21. A user updates their profile picture, refreshes, and sees the old one. Then the new one. Then the old one. What's happening and how do you fix it?

---

## 07 · Clock Synchronization

- [ ] 🟢 Q22. Why can't we trust timestamps across machines?
- [ ] 🟡 Q23. What's the difference between a wall clock and a monotonic clock?
- [ ] 🔴 Q24. Your multi-region store uses last-write-wins by timestamp and users report edits disappearing. Why?

---

## 08 · Logical Clocks

- [ ] 🟢 Q25. What does "happened-before" mean?

---

## 09 · Lamport Timestamps

- [ ] 🟡 Q26. How do Lamport timestamps work and what's their limitation?

---

## 10 · Vector Clocks

- [ ] 🟡 Q27. How do vector clocks detect conflicts?

---

## 11 · Consensus

- [ ] 🟢 Q28. What problem does consensus solve?
- [ ] 🟡 Q29. Why does consensus use a majority quorum?
- [ ] 🟡 Q30. What does the FLP result say?
- [ ] 🔴 Q31. A teammate wants to store all user sessions in etcd "because it's strongly consistent". What do you say?

---

## 12 · Paxos

- [ ] 🟡 Q32. Explain Paxos at a high level.

---

## 13 · Raft

- [ ] 🟢 Q33. What are the roles and terms in Raft?
- [ ] 🟡 Q34. Walk through a Raft leader election.
- [ ] 🔴 Q35. A Raft leader gets a write, stores it, and crashes before replicating. Is the write lost?

---

## 14 · Leader Election

- [ ] 🟢 Q36. Why do systems elect a leader?
- [ ] 🟡 Q37. How would you implement leader election for three instances of a cron service?
- [ ] 🔴 Q38. After a leader's long GC pause, users got duplicate emails. Why, and how do you fix it?

---

## 15 · Distributed Locks

- [ ] 🟢 Q39. How do you build a simple lock in Redis?
- [ ] 🟡 Q40. What's the Redlock debate about?
- [ ] 🔴 Q41. Two workers both processed the same payment despite a distributed lock. How?

---

## 16 · Gossip Protocol

- [ ] 🟡 Q42. How does a gossip protocol work and where is it used?

---

## 17 · CRDTs

- [ ] 🟢 Q43. What is a CRDT?
- [ ] 🟡 Q44. How would you build a like counter that works across 3 active regions?
- [ ] 🔴 Q45. Could you use a CRDT for a bank account balance that must never go negative?

---

## 18 · Operational Transformation

- [ ] 🟡 Q46. What is Operational Transformation?
- [ ] 🔴 Q47. You're designing a collaborative editor that must work offline. OT or CRDT?
