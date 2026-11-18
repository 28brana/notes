# System Design · Distributed Systems

[← Back to System Design](../README.md) · [Interview Q&A →](questions/distributed-systems-qa.md) · [Practice questions 📝](questions/practice-questions.md)

What goes wrong when many machines must act as one — partial failures, partitions, drifting clocks — and the tools (consensus, leader election, locks, CRDTs) that make it work. Interviewers use these to test senior-level depth.

| # | Topic | In one line | Priority |
|---|---|---|---|
| 01 | [Challenges of Distribution](01-challenges-of-distribution.md) | Partial failure, unreliable network and clocks, the 8 fallacies | 🔥 High |
| 02 | [Network Partitions](02-network-partitions.md) | Nodes alive but can't talk — majority side keeps writing | ⚡ Medium |
| 03 | [Split Brain](03-split-brain.md) | Two leaders at once — fix with quorum + fencing | ⚡ Medium |
| 04 | [Heartbeats](04-heartbeats.md) | Periodic "I'm alive" messages to suspect failures | ⚡ Medium |
| 05 | [Handling Failures](05-handling-failures.md) | Timeouts, backoff + jitter, idempotency, fencing tokens | 🔥 High |
| 06 | [Consistency Models](06-consistency-models.md) | Linearizable → causal → eventual, plus read-your-writes | 🔥 High |
| 07 | [Clock Synchronization](07-clock-synchronization.md) | Clocks drift; NTP is approximate; don't order by wall time | ⚡ Medium |
| 08 | [Logical Clocks](08-logical-clocks.md) | Order events by causality, not real time | 💤 Low |
| 09 | [Lamport Timestamps](09-lamport-timestamps.md) | One counter, max + 1 on receive — total order | 💤 Low |
| 10 | [Vector Clocks](10-vector-clocks.md) | Counter per node — detects concurrent writes | 💤 Low |
| 11 | [Consensus](11-consensus.md) | Majority agrees on one value / log — etcd, ZooKeeper | 🔥 High |
| 12 | [Paxos](12-paxos.md) | Classic two-phase consensus — prepare, accept | 💤 Low |
| 13 | [Raft](13-raft.md) | Leader + terms + majority-committed log | ⚡ Medium |
| 14 | [Leader Election](14-leader-election.md) | Pick one node in charge via leases + epochs | 🔥 High |
| 15 | [Distributed Locks](15-distributed-locks.md) | Redis SET NX vs etcd/ZooKeeper, fencing tokens | ⚡ Medium |
| 16 | [Gossip Protocol](16-gossip-protocol.md) | Rumor-style spreading of cluster state in log N rounds | 💤 Low |
| 17 | [CRDTs](17-crdts.md) | Data types that always merge without conflicts | ⚡ Medium |
| 18 | [Operational Transformation](18-operational-transformation.md) | Transform concurrent edits — Google Docs vs CRDTs | ⚡ Medium |
