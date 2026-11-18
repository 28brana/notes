# Q&A · Distributed Systems

[← Back to Distributed Systems](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 01 · Challenges of Distribution

[Read the notes →](../01-challenges-of-distribution.md)

> [!question]- 🟢 Q1. What makes distributed systems harder than a single machine?
> Partial failure: some parts fail while others keep running, and you often can't tell if a node is dead or just slow. On top of that the network loses, delays and reorders messages, and clocks on different machines disagree. There's no shared memory and no shared clock.

> [!question]- 🟡 Q2. Name some of the fallacies of distributed computing.
> The network is reliable, latency is zero, bandwidth is infinite, the network is secure, topology doesn't change, there's one administrator, transport cost is zero, and the network is homogeneous. The first three cause the most bugs — chatty services and missing timeouts come straight from them.

> [!question]- 🟡 Q3. A request to another service timed out. What do you know about it?
> Almost nothing. The request might never have arrived, the server might have crashed mid-way, or it might have finished and only the reply was lost. That's why retries must be idempotent and why a timeout never proves the work didn't happen.

> [!question]- 🔴 Q4. You moved from 10 to 1,000 servers and now see "random" failures daily. Why, and how do you think about it?
> At that scale, hardware and network faults are constant — if each machine fails every few years, one dies about every day. Failure is the normal state, not an edge case. I'd design for it: redundancy, timeouts, idempotent retries, failure detection and automatic recovery, and good monitoring to see patterns.

---

## 02 · Network Partitions

[Read the notes →](../02-network-partitions.md)

> [!question]- 🟢 Q5. What is a network partition?
> The network splits the cluster into groups that can't talk to each other, but every node is still alive and serving clients. Each side may think the other died. It can be a full split, partial, one-way, or just extreme slowness.

> [!question]- 🟡 Q6. Why do clusters use an odd number of nodes?
> Because decisions need a majority. With 5 nodes, any split leaves one side with at least 3, so it can keep going. With 4 nodes, a 2–2 split leaves no majority and nobody can write — and 4 tolerates only one failure, the same as 3.

> [!question]- 🔴 Q7. The link between your two data centers drops. DC-A has 3 DB nodes, DC-B has 2. What should happen?
> In a CP setup, DC-A has the majority so it keeps accepting writes; DC-B's nodes step down or go read-only and return errors for writes. When the link heals, DC-B catches up from DC-A's log. If the business needs writes in both DCs, we'd choose an AP design and plan conflict merging.

---

## 03 · Split Brain

[Read the notes →](../03-split-brain.md)

> [!question]- 🟢 Q8. What is split brain?
> Two nodes both believe they're the leader and both accept writes. Their data diverges, and when they reconnect you have conflicting or lost updates. It usually happens after a partition or a long pause on the old leader.

> [!question]- 🟡 Q9. How do you prevent split brain?
> Use a majority quorum so only one side of a partition can elect a leader. Add fencing tokens — each leader gets a higher epoch, and storage rejects writes with an older one — so a stale leader can't do damage. For two-node setups, add a witness node or use STONITH.

> [!question]- 🔴 Q10. You have a primary and one standby with automatic failover, and twice this month you had two primaries. What's wrong?
> A two-node cluster can't tell "the other node died" from "the link died", so the standby promotes itself during a blip. Fix it by adding a third witness or quorum service so failover needs a majority, and add fencing so the old primary's writes are rejected once a new one exists.

---

## 04 · Heartbeats

[Read the notes →](../04-heartbeats.md)

> [!question]- 🟢 Q11. What is a heartbeat and why is it used?
> A small periodic "I'm alive" message between nodes, for example every second. If a node misses several in a row, others suspect it has failed and trigger failover. It's how clusters detect crashed nodes, since a dead node can't report its own death.

> [!question]- 🟡 Q12. How do you choose the heartbeat timeout?
> It's a trade-off. Short timeouts give fast failover but cause false alarms on GC pauses or network blips; long timeouts avoid flapping but leave users waiting. I'd base it on observed latency, use a few missed beats, or use an adaptive detector like phi accrual.

> [!question]- 🔴 Q13. Your cluster keeps electing a new leader every few minutes even though nothing crashed. Why?
> The heartbeat timeout is probably too short for real conditions — GC pauses, overloaded nodes, or network jitter delay heartbeats past the timeout. Check if heartbeats share a busy thread or queue with real work. Fix by raising the timeout, prioritizing heartbeat traffic, and fixing the pauses.

---

## 05 · Handling Failures

[Read the notes →](../05-handling-failures.md)

> [!question]- 🟢 Q14. Why does every network call need a timeout?
> Without one, a slow or dead dependency holds your threads forever, and soon your whole service is stuck. A timeout based on the dependency's p99 latency frees resources and lets you retry or fall back.

> [!question]- 🟡 Q15. What are exponential backoff and jitter, and why both?
> Backoff doubles the wait between retries — 100, 200, 400 ms — so a struggling service gets room to recover. Jitter adds randomness so thousands of clients don't retry at the same instant and create a thundering herd. Cap attempts at 3–5 and only retry transient errors.

> [!question]- 🟡 Q16. What is a fencing token?
> An increasing number handed out each time a lock or leadership is granted. The storage remembers the highest token it has seen and rejects writes with lower ones. It stops a "zombie" node that paused and lost its lock from overwriting newer data.

> [!question]- 🔴 Q17. During an outage, your downstream service got 27 times its normal traffic. What happened?
> Retries at every layer multiplied: if three layers each retry three times, the bottom service sees 3 × 3 × 3 = 27 calls per user request. Fix by retrying at only one layer, adding backoff with jitter, a retry budget, and a circuit breaker so callers stop hammering a service that's clearly down.

---

## 06 · Consistency Models

[Read the notes →](../06-consistency-models.md)

> [!question]- 🟢 Q18. What is eventual consistency?
> If writes stop, all replicas will eventually hold the same value — but there's no promise on how soon, and reads in between may be stale. It's cheap and highly available, fine for likes, feeds and counters.

> [!question]- 🟡 Q19. What's the difference between linearizable and causal consistency?
> Linearizable makes the system act like a single copy: once a write completes, every read sees it. Causal only promises that related events appear in cause-then-effect order, like a post before its reply; unrelated writes can appear in different orders on different replicas. Causal is cheaper and stays available during partitions.

> [!question]- 🟡 Q20. What is read-your-writes consistency and how do you implement it?
> A user always sees their own changes, even if others see them later. Implement it by reading that user's own data from the leader for a short window after a write, or by tracking the write's version and only reading from replicas that have caught up.

> [!question]- 🔴 Q21. A user updates their profile picture, refreshes, and sees the old one. Then the new one. Then the old one. What's happening and how do you fix it?
> Their reads hit different replicas with different replication lag — that breaks read-your-writes and monotonic reads. Fix by routing the user's reads of their own profile to the leader for a minute after the edit, and pinning each user's reads to one replica so time never goes backward.

---

## 07 · Clock Synchronization

[Read the notes →](../07-clock-synchronization.md)

> [!question]- 🟢 Q22. Why can't we trust timestamps across machines?
> Each machine's clock drifts, and NTP only keeps them within a few milliseconds — sometimes it even jumps a clock backward. So a later event on one machine can get an earlier timestamp than an event on another.

> [!question]- 🟡 Q23. What's the difference between a wall clock and a monotonic clock?
> The wall clock is time-of-day, good for display and logs, but it can jump when synced. The monotonic clock only moves forward and is meant for measuring durations and timeouts on one machine; it's meaningless to compare across machines.

> [!question]- 🔴 Q24. Your multi-region store uses last-write-wins by timestamp and users report edits disappearing. Why?
> Clock skew: a node whose clock runs behind gives a newer write an older timestamp, so LWW throws it away. Fix by using logical clocks or version vectors to detect concurrent writes, merging with CRDTs, or routing writes for a key through one leader.

---

## 08 · Logical Clocks

[Read the notes →](../08-logical-clocks.md)

> [!question]- 🟢 Q25. What does "happened-before" mean?
> A happened-before B if they're on the same node and A came first, if A sends a message that B receives, or through a chain of those. If neither happened-before the other, they're concurrent — neither could have known about the other.

---

## 09 · Lamport Timestamps

[Read the notes →](../09-lamport-timestamps.md)

> [!question]- 🟡 Q26. How do Lamport timestamps work and what's their limitation?
> Each node keeps a counter, increments it on each event, attaches it to messages, and on receive sets it to max(local, received) + 1. Ties are broken by node ID for a total order. The limitation: a smaller timestamp doesn't prove causality — it can't detect concurrent events.

---

## 10 · Vector Clocks

[Read the notes →](../10-vector-clocks.md)

> [!question]- 🟡 Q27. How do vector clocks detect conflicts?
> Each value carries one counter per node. If one vector is less than or equal to the other in every slot, it's older. If each is ahead in some slot, the writes were concurrent — a real conflict that must be merged by the app or a CRDT.

---

## 11 · Consensus

[Read the notes →](../11-consensus.md)

> [!question]- 🟢 Q28. What problem does consensus solve?
> Getting a group of nodes to agree on one value — or one order of operations — even when some crash or messages are lost. It's the basis for leader election, distributed locks, config stores and strongly consistent databases.

> [!question]- 🟡 Q29. Why does consensus use a majority quorum?
> Any two majorities overlap in at least one node, so a new decision always sees the previous one and two conflicting decisions can't both win. With 2f+1 nodes you tolerate f failures — 5 nodes survive 2.

> [!question]- 🟡 Q30. What does the FLP result say?
> In a fully asynchronous network where even one node can crash, no algorithm can guarantee consensus finishes in bounded time. Practical algorithms like Raft stay always safe and use timeouts to make progress when the network behaves.

> [!question]- 🔴 Q31. A teammate wants to store all user sessions in etcd "because it's strongly consistent". What do you say?
> etcd is built for small, critical metadata — every write goes through a leader and a majority round trip, so it won't handle high-volume session traffic well. Sessions need speed more than consensus; put them in Redis or a regular DB, and keep etcd for config, leader election and locks.

---

## 12 · Paxos

[Read the notes →](../12-paxos.md)

> [!question]- 🟡 Q32. Explain Paxos at a high level.
> A proposer picks a unique number and asks a majority of acceptors to promise to ignore lower numbers, learning any value already accepted. Then it asks them to accept a value — reusing the earlier one if there was one. Once a majority accepts, the value is chosen. Multi-Paxos adds a stable leader so phase one is skipped for each new entry.

---

## 13 · Raft

[Read the notes →](../13-raft.md)

> [!question]- 🟢 Q33. What are the roles and terms in Raft?
> Nodes are followers, candidates or the leader. Time is split into numbered terms, with at most one leader per term; messages from older terms are ignored, so the term works as a fencing token.

> [!question]- 🟡 Q34. Walk through a Raft leader election.
> A follower that doesn't hear a heartbeat within its random election timeout becomes a candidate, increments the term, votes for itself and asks for votes. Nodes vote once per term and only for candidates whose log is at least as up-to-date. A majority makes it leader; random timeouts avoid repeated split votes.

> [!question]- 🔴 Q35. A Raft leader gets a write, stores it, and crashes before replicating. Is the write lost?
> Yes, and that's correct behaviour — it was never committed because a majority never stored it, so the client never got a success reply. The new leader won't have it, and when the old node rejoins its uncommitted entry is overwritten. The client will time out and should retry with an idempotency key.

---

## 14 · Leader Election

[Read the notes →](../14-leader-election.md)

> [!question]- 🟢 Q36. Why do systems elect a leader?
> One leader makes coordination simple: one writer means no conflicts, one scheduler means a job runs once, one coordinator assigns work. The cost is a write bottleneck and a short gap during failover.

> [!question]- 🟡 Q37. How would you implement leader election for three instances of a cron service?
> Each instance tries to create a leader key with a lease in etcd or ZooKeeper; the winner runs the jobs and keeps renewing. Others watch the key and race again if the lease expires. Each new leader gets a higher epoch, which it passes along as a fencing token.

> [!question]- 🔴 Q38. After a leader's long GC pause, users got duplicate emails. Why, and how do you fix it?
> The lease expired during the pause, a new leader was elected and started sending, then the old leader woke up still believing it was leader. Fix with fencing tokens checked by the downstream (or an idempotency key per email), and have the leader check its lease right before every action.

---

## 15 · Distributed Locks

[Read the notes →](../15-distributed-locks.md)

> [!question]- 🟢 Q39. How do you build a simple lock in Redis?
> `SET key random-value NX PX 30000` — NX only sets it if it doesn't exist, and PX makes it expire so a crashed holder doesn't block forever. Release with a Lua script that deletes only if the value is still yours.

> [!question]- 🟡 Q40. What's the Redlock debate about?
> Redlock takes the lock on a majority of independent Redis nodes. Critics, notably Martin Kleppmann, argued it relies on bounded clock drift and pauses, so a paused client can keep working after its lock expired — unsafe for correctness. It's fine for efficiency; for correctness use a consensus store plus fencing tokens.

> [!question]- 🔴 Q41. Two workers both processed the same payment despite a distributed lock. How?
> Likely worker A's lock expired while it was paused or slow, worker B acquired it, and then A resumed and finished anyway. A TTL lock alone can't stop that. Fix with fencing tokens checked by the payment store, plus an idempotency key on the payment so a duplicate is rejected.

---

## 16 · Gossip Protocol

[Read the notes →](../16-gossip-protocol.md)

> [!question]- 🟡 Q42. How does a gossip protocol work and where is it used?
> Every second or so each node shares its view with a few random peers, and they merge the newer info. News spreads in about log N rounds with no central coordinator. Cassandra, Consul and Redis Cluster use it for membership and failure detection — but it's eventually consistent, so not for leader election or locks.

---

## 17 · CRDTs

[Read the notes →](../17-crdts.md)

> [!question]- 🟢 Q43. What is a CRDT?
> A data type whose merge is commutative, associative and idempotent, so replicas can accept writes independently — even offline — and always converge to the same state without coordination.

> [!question]- 🟡 Q44. How would you build a like counter that works across 3 active regions?
> Use a G-Counter, or a PN-Counter if unlikes exist: each region keeps its own count, the total is the sum, and merging takes the max per region. No likes are lost, unlike last-write-wins on a single number.

> [!question]- 🔴 Q45. Could you use a CRDT for a bank account balance that must never go negative?
> Not on its own. CRDTs merge without coordination, so two regions could each approve a withdrawal that's fine locally but overdraws the account once merged. Rules like "balance ≥ 0" or "unique username" need coordination — a single leader or consensus — for those operations.

---

## 18 · Operational Transformation

[Read the notes →](../18-operational-transformation.md)

> [!question]- 🟡 Q46. What is Operational Transformation?
> When two users edit at the same time, OT transforms each incoming edit against the concurrent ones already applied so positions still line up — for example, shifting an insert left after an earlier delete. Usually a central server sets the order. Google Docs uses it.

> [!question]- 🔴 Q47. You're designing a collaborative editor that must work offline. OT or CRDT?
> CRDT. OT relies on a central server to order operations and gets complicated when clients diverge for a long time. A CRDT like Yjs or Automerge lets each client edit offline and merge on reconnect, at the cost of extra metadata per character. I'd still use WebSockets to sync live when online.
