# Q&A · Consistent Hashing

[← Back to Core Concepts](../README.md) · [Read the notes →](../06-consistent-hashing.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

> [!question]- 🟢 Q1. What is consistent hashing, in one or two sentences?
> It's a way to map keys to servers where both are hashed onto the same ring, and each key goes to the next server clockwise. When a server joins or leaves, only about 1/N of the keys move instead of almost all of them.

> [!question]- 🟢 Q2. What's wrong with `hash(key) % N`?
> N is part of the formula, so when N changes almost every key maps to a different node. Going from 4 to 5 nodes moves about 80% of keys, and from N to N+1 it moves N/(N+1). For a cache that's a miss storm on the database; for a database it means copying most of the data.

> [!question]- 🟢 Q3. How does the hash ring decide which server owns a key?
> Hash every server onto a ring from 0 to 2³² − 1, and hash the key onto the same ring. Starting at the key's position, walk clockwise; the first server you hit owns the key. So each server owns the arc between the previous server and itself.

> [!question]- 🟢 Q4. What happens when you add or remove a server on the ring?
> Adding a server only takes the keys in the arc just before it, from its clockwise neighbour — about 1/N of the keys. Removing one hands its keys to the next server clockwise. Every other key stays where it was.

> [!question]- 🟢 Q5. What are virtual nodes and why do we need them?
> Each physical server is placed on the ring many times, say 100–200 points, instead of once. With only one point per server, the arcs are random sizes, so load is lumpy. Vnodes average that out, and when a server dies its many small arcs spread across many servers instead of dumping everything on one neighbour.

> [!question]- 🟡 Q6. How do you handle servers with different capacities?
> Give them vnodes in proportion to their capacity. A server with twice the RAM gets twice as many vnodes, say 400 instead of 200, so it owns roughly twice the keys. Removing vnodes is also a gentle way to drain a server before taking it out.

> [!question]- 🟡 Q7. How is replication done on a consistent hash ring?
> Store the key on the owner plus the next R − 1 distinct physical servers clockwise — Dynamo calls this the preference list. You must skip vnodes that belong to a server you've already picked, and ideally servers in the same rack or availability zone, so one failure can't take out every copy.

> [!question]- 🟡 Q8. How would you implement the lookup? What's the complexity?
> Keep a sorted array of all vnode hashes with their servers. To look up a key, hash it and binary-search for the first vnode hash that's greater or equal, wrapping to index 0 if you fall off the end. That's O(log V) — about 15 steps for 20,000 vnodes.

> [!question]- 🟡 Q9. Does consistent hashing solve hot keys?
> No. It spreads keys evenly, not traffic. A single celebrity post getting 100,000 req/s still lives on one node. You need extra tricks: a local in-memory cache on app servers, splitting the key into copies like `post:9001#1..#10`, reading from all replicas, and request coalescing.

> [!question]- 🟡 Q10. How do all clients agree on what the ring looks like?
> They need the same server list. Options are a config service like ZooKeeper or etcd that pushes membership changes, a gossip protocol like Cassandra uses, or a proxy layer that does routing so clients don't need the ring at all. If two clients disagree, they'll read and write the same key on different nodes.

> [!question]- 🟡 Q11. Name real systems that use consistent hashing.
> Amazon Dynamo and DynamoDB, Cassandra, Riak, memcached clients using the ketama algorithm, Discord for routing guilds to processes, load balancers like Google Maglev and Envoy, and CDNs like Akamai so the same URL hits the same cache server.

> [!question]- 🟡 Q12. How is Redis Cluster's approach different from a hash ring?
> Redis Cluster uses a fixed 16,384 hash slots: `slot = CRC16(key) % 16384`, and each node is assigned a set of slots. Since the slot count never changes, the key-to-slot mapping is stable. Scaling means explicitly moving some slots to the new node — it's a table you control, rather than ownership decided by positions on a ring.

> [!question]- 🔴 Q13. We added a 5th Redis node and the database immediately went to 100% CPU. What happened and how do you fix it?
> The client was almost certainly routing with `hash(key) % N`, so going from 4 to 5 nodes remapped about 80% of keys and caused millions of cache misses at once. Short term: roll back to 4 nodes and add request coalescing so a miss storm can't stampede the DB. Long term: switch to a consistent hashing client with vnodes, or Redis Cluster slots, so adding a node only moves about 20% of keys — and add nodes during quiet hours.

> [!question]- 🔴 Q14. One cache node died, and then its neighbour died a few minutes later, and then the next one. Why?
> That's a cascading failure from a ring without vnodes, or with too few. All of the dead node's keys moved to its single clockwise neighbour, doubling its load until it fell over, and then the same happened to the next one. Fix it with 100–200 vnodes per server so a failed node's load spreads across all the others, plus spare capacity and rate limiting on misses.

> [!question]- 🔴 Q15. A celebrity posts and one cache node hits 100% CPU while the others are idle. How do you handle it?
> Consistent hashing put that one hot key on one node, and adding nodes won't help. I'd cache the post in each app server's memory for a few seconds, split the key into N copies on different nodes and pick one at random on read, and coalesce concurrent misses. I'd also add hot-key detection so this kicks in automatically next time.

> [!question]- 🔴 Q16. When would you choose rendezvous hashing or jump hash instead of a ring?
> Rendezvous (HRW) scores every server for each key and picks the highest — perfectly even with no vnodes, but O(N) per lookup, so it suits small clusters of tens of servers. Jump hash needs no memory and balances almost perfectly, but you can only add or remove buckets at the end, so it fits numbered storage shards, not servers that fail at random. For a large dynamic cluster, a ring with vnodes or fixed slots is usually the practical choice.
