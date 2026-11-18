# 06 · Consistency Models

> [!info] Topic info
> **Subject:** System Design · **Chapter:** Distributed Systems · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Distributed Systems](README.md) · [Interview questions →](questions/distributed-systems-qa.md)

> [!abstract] In one line
> A **consistency model** is the **promise** a replicated system makes about **what a read can return** after a write — from **strong** (always the latest) to **eventual** (latest, someday).

---

## 1. The problem

Our app has the posts DB replicated to 3 regions. A user edits their bio, refreshes, and sees the **old** bio. Then refreshes again and sees the new one. Then the old one again.

Is that a bug? It depends on what the system **promised**. That promise is the consistency model.

> [!note] Recap
> [CAP Theorem](../core-concepts/07-cap-theorem.md) introduced the spectrum (strong → eventual). Here we define each level properly and add the **client-centric** guarantees interviewers ask about.

---

## 2. The main models (strongest → weakest)

| Model | Promise | Cost | Example |
|---|---|---|---|
| **Linearizable (strong)** | Acts like **one copy**. Once a write finishes, every read sees it. | Slow, needs consensus/quorum, unavailable in partitions | etcd, Spanner, ZooKeeper writes |
| **Sequential** | Everyone sees all ops in the **same order**, but it may lag real time | Still needs coordination | Rarely named directly |
| **Causal** | If A **caused** B (reply after a post), everyone sees A before B. Unrelated ops can be in any order. | Moderate — track dependencies | Some geo-DBs, COPS research |
| **Eventual** | If writes stop, replicas **eventually** agree. No ordering promise. | Cheapest, always available | DNS, Cassandra (ONE), DynamoDB default reads |

```mermaid
flowchart LR
    L[🔒 Linearizable] --> S[📋 Sequential]
    S --> C[🔗 Causal]
    C --> E[🌊 Eventual]
```

Left = easier to program, slower. Right = faster and more available, but the app must handle stale data.

> [!example] Causal in our app
> Alice posts "I got the job!" and Bob comments "Congrats!". With **eventual**, Carol might see Bob's "Congrats!" with no post. With **causal**, the comment never shows before the post it replies to.

---

## 3. Client-centric guarantees (session guarantees)

Often you don't need global strong consistency — you just need **one user's** view to make sense.

| Guarantee | Promise | Fixes |
|---|---|---|
| **Read-your-writes** | You always see your **own** writes | "I edited my bio and it reverted" |
| **Monotonic reads** | You never see data go **back in time** | Refresh shows new, then old, then new |
| **Monotonic writes** | Your writes apply in the order you made them | Edit 2 overwritten by edit 1 |
| **Consistent prefix** | You see writes in an order that makes sense | Answer appears before the question |

**How to get them cheaply:**
- Read-your-writes → read **your own profile from the leader** for ~1 minute after you edit it, or track the write's version and only read replicas that have caught up.
- Monotonic reads → **stick each user to one replica** (hash user ID).

> [!tip] 🎤 Say it like this
> "Consistency models are a contract about what reads can return. Linearizable means it acts like a single copy — great for locks, balances, and unique usernames, but costly. Most feeds and like counts are fine with eventual consistency. And often the real need is just read-your-writes for the user who made the change, which I get by routing their reads to the leader for a short time."

---

## 4. Choosing per feature

| Feature in our app | Model |
|---|---|
| **Username signup, payments** | Linearizable |
| **Comments under a post** | Causal |
| **Editing your own profile** | Read-your-writes |
| **Like counts, follower counts, feed** | Eventual |

> [!failure] Interview traps
> - Mixing up **CAP's C** (linearizability) with **ACID's C** (data obeys rules/constraints). Totally different.
> - "Eventual consistency means data might never sync" — no, it **will** converge once writes stop; there's just no time bound.
> - Picking strong consistency for everything — you pay latency on every request for features that don't need it.

---

## Summary

- **Linearizable** > **Sequential** > **Causal** > **Eventual** — stronger is simpler to reason about but slower.
- **Session guarantees** (read-your-writes, monotonic reads) fix most user-visible weirdness cheaply.
- Choose **per feature**, not per system.

> [!success] Key takeaway
> Ask "what's the worst thing that happens if this read is stale?" — that answer picks your consistency model.
