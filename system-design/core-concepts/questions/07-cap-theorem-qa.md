# Q&A · CAP Theorem

[← Back to Core Concepts](../README.md) · [Read the notes →](../07-cap-theorem.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

> [!question]- 🟢 Q1. What is the CAP theorem?
> In a distributed system, when the network between nodes breaks, you can't have both consistency and availability. You either refuse some requests to keep all copies in agreement, or keep answering everyone and accept the copies may disagree for a while. When there's no partition, you can have both.

> [!question]- 🟢 Q2. What do C, A, and P mean exactly?
> C is linearizability — every read sees the latest successful write, as if there's one copy. A means every request to a non-failed node gets a non-error response. P means the system keeps working even when messages between nodes are lost or delayed.

> [!question]- 🟢 Q3. Is CAP's C the same as the C in ACID?
> No. ACID's C means the data follows its rules — constraints like `balance >= 0` or valid foreign keys. CAP's C means all replicas agree and every read returns the latest write. Same letter, different idea.

> [!question]- 🟢 Q4. Why can't you just "pick any 2 of 3"?
> Because in a distributed system you don't get to opt out of partitions — cables get cut, switches fail, nodes pause. So P is a given. The real choice is C or A while a partition is happening.

> [!question]- 🟢 Q5. Is a CA system possible?
> Only on a single machine, like one Postgres server with no replicas — there's no network between copies to break. As soon as you have two nodes over a network, you must decide what happens during a partition, so CA isn't a real option for distributed systems.

> [!question]- 🟡 Q6. Is CAP availability the same as 99.99% uptime?
> No. Uptime nines measure how often the service works overall. CAP availability is a strict rule that every working node must answer every request, even when cut off. A CP database like etcd can still hit 99.99% uptime because partitions are rare and the majority side keeps working.

> [!question]- 🟡 Q7. Walk me through what a CP system does during a partition.
> Say 3 nodes, 2 in Mumbai and 1 in Singapore, and writes need a majority. When the link breaks, Mumbai has 2 of 3 and keeps accepting writes. Singapore can't reach a majority, so it rejects writes or times out, and may refuse reads too. No conflicts — but Singapore users see errors until the link heals.

> [!question]- 🟡 Q8. And what does an AP system do?
> Both sides keep accepting reads and writes locally. Users get answers, but each side may show different data — Mumbai shows 1,050 likes, Singapore 1,030. When the link comes back, the copies sync and resolve conflicts, ending at 1,080. That's eventual consistency.

> [!question]- 🟡 Q9. In a social media app, which features would you make CP and which AP?
> AP for likes, view counts, feed, comments and notifications — a few seconds stale hurts no one. CP for payments, wallet balance, unique usernames, inventory, and security actions like bans or password changes — an error is better than a wrong answer there. CAP is a per-feature choice, not one label for the whole app.

> [!question]- 🟡 Q10. Name some CP-leaning and AP-leaning databases.
> CP-leaning: ZooKeeper, etcd, HBase, MongoDB with default settings, and Spanner. AP-leaning: Cassandra, DynamoDB, Riak, CouchDB. Many are tunable — Cassandra and DynamoDB can do strongly consistent reads if you ask for them.

> [!question]- 🟡 Q11. What does R + W > N mean?
> N is the number of copies, W is how many must confirm a write, R is how many you read from. If R + W > N, the read set and write set always overlap, so a read hits at least one copy with the latest write. With N = 3, W = 2 and R = 2 gives that; W = 1 and R = 1 is faster but can be stale.

> [!question]- 🟡 Q12. What consistency models sit between strong and eventual?
> Causal — effects never show before their causes, like a reply before its comment. Read-your-writes — you always see your own changes. Monotonic reads — you never go back in time on refresh. These are cheap to add on top of an AP system and make it feel consistent to users.

> [!question]- 🔴 Q13. How do AP systems resolve conflicts, and what's the danger with last-write-wins?
> Last-write-wins keeps the write with the newest timestamp and drops the other — but clocks drift, so a later write can lose, and nobody gets an error. Vector clocks detect truly concurrent writes and keep both so the app can merge. CRDTs, like a per-region like counter, merge automatically with no data loss.

> [!question]- 🔴 Q14. What is PACELC and why does it matter?
> CAP only covers partitions, which are rare. PACELC says: if Partition, choose Availability or Consistency; Else, choose Latency or Consistency. Strong consistency means waiting for replicas — ~60 ms per write between Mumbai and Singapore — even on a healthy day. DynamoDB is PA/EL, Spanner is PC/EC.

> [!question]- 🔴 Q15. Scenario: during a network split, two users in different regions both got the same username. What went wrong and how do you fix it?
> The username service was AP — each region accepted the write locally without checking the other. Uniqueness needs CP: route username claims through a single leader or a consensus-based store like etcd or Spanner, so only the majority side can accept them. The minority region returns "try again later" for username changes during the split, which is fine because it's rare.

> [!question]- 🔴 Q16. Scenario: users complain their like count jumps 120 → 98 → 120 on refresh, and after editing their bio they still see the old one. How do you fix it without going fully CP?
> Their reads are hitting different replicas that lag behind. For the jumping count, give monotonic reads by sticking each user to one replica, e.g. by hashing their user ID. For the bio, give read-your-writes by sending that user's reads to the leader for ~10 seconds after they write. The system stays AP and fast, but feels consistent.

> [!question]- 🔴 Q17. How would you bring up CAP in a system design interview?
> I'd say partitions will happen, so I choose C or A per feature. I'd list what can be stale versus what must be exact, then say what each side does during a split — e.g. payments error on the minority side, likes keep working and merge with CRDTs. Then I'd mention PACELC: I only pay the cross-region latency of strong consistency where it's truly needed.
