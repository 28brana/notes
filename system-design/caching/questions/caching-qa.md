# Q&A · Caching

[← Back to Caching](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 01 · What Is Caching

[Read the notes →](../01-what-is-caching.md)

> [!question]- 🟢 Q1. What is a cache, and why does it make a system faster?
> A cache is a small, fast store (usually RAM) that holds a copy of data we read often. Reads hit the cache in ~1 ms instead of the database in ~50 ms. It works because traffic is skewed — a small set of hot items gets most of the reads.

> [!question]- 🟢 Q2. What is a cache hit ratio, and why does going from 90% to 99% matter so much?
> Hit ratio is hits divided by total reads. At 90% the DB still gets 10% of all reads; at 99% it gets 1% — ten times less load. Average latency also drops sharply because misses are the expensive part.

> [!question]- 🟡 Q3. Name the levels where data can be cached between a user and the database.
> Browser cache, CDN edge, in-memory cache inside each app server, a distributed cache like Redis, and the database's own buffer pool. The closer to the user, the faster — but the harder to invalidate.

> [!question]- 🔴 Q4. What data would you NOT cache in a payments system, and why?
> Account balances and anything used to decide whether to allow a payment, because a stale value could allow overspending. I'd cache things like merchant details or currency lists that change rarely. The rule: only cache data that can be a little stale without causing harm.

---

## 02 · Caching Strategies

[Read the notes →](../02-caching-strategies.md)

> [!question]- 🟢 Q5. Explain the cache-aside pattern.
> The app checks the cache first. On a miss it reads the database, writes the result into the cache with a TTL, and returns it. On writes, it updates the database and deletes the cache key. It's the most common pattern because it's simple and survives a cache outage.

> [!question]- 🟡 Q6. What's the difference between write-through, write-behind and write-around?
> Write-through writes the cache and DB together — fresh reads, slower writes. Write-behind writes only the cache and flushes to the DB later — fastest, but can lose data if the cache crashes. Write-around writes only the DB, so the cache isn't filled with data nobody reads.

> [!question]- 🟡 Q7. When is write-behind a good idea, and when is it dangerous?
> Good for high-volume, low-value writes like view counts or likes, where batching 1,000 updates into one DB write saves a lot. Dangerous for orders or payments, because writes that were acknowledged to the user can vanish if the cache dies before flushing.

> [!question]- 🔴 Q8. Users edit their profile but still see the old version right after saving. Your setup is cache-aside with a 1-hour TTL. What's wrong and how do you fix it?
> The write updates the DB but nobody clears the cache, so reads serve the old copy until the TTL ends. Fix: after the DB write, delete the cache key so the next read reloads fresh data. If read-your-own-write must be instant, use write-through for that key.

---

## 03 · Cache Eviction Policies

[Read the notes →](../03-cache-eviction-policies.md)

> [!question]- 🟢 Q9. What's the difference between eviction and expiration?
> Eviction removes keys because the cache is full and needs room. Expiration removes keys because their TTL ran out. They are separate and usually used together.

> [!question]- 🟡 Q10. Compare LRU and LFU. When would LFU beat LRU?
> LRU evicts the key unused for the longest time; LFU evicts the key used the fewest times. LFU wins when popularity is stable and you have one-off scans — a batch job reading every product once would flush the hot keys under LRU, but not under LFU.

> [!question]- 🟡 Q11. How would you implement an LRU cache with O(1) get and put?
> A hash map from key to node, plus a doubly linked list ordered by use, most recent at the head. On get, find the node in the map and move it to the head. On put, insert at the head and, if over capacity, remove the tail node and its map entry. Doubly linked so we can unlink a node in O(1).

> [!question]- 🔴 Q12. After a nightly report job runs, the morning cache hit ratio drops from 95% to 60%. Why?
> The job reads lots of old, cold data once, and under LRU each of those reads pushes a hot key out. In the morning, the real hot set has been evicted. Fixes: have the job bypass the cache or read from a replica, or switch to an LFU-style policy that resists scans.

---

## 04 · Cache Invalidation

[Read the notes →](../04-cache-invalidation.md)

> [!question]- 🟢 Q13. What are the main ways to invalidate a cache?
> TTL expiry, deleting the key when the data is written, versioned keys where an update switches to a new key, and event-driven invalidation where a change event triggers deletes. Most systems combine delete-on-write with a TTL as a safety net.

> [!question]- 🟡 Q14. On a write, why delete the cache key instead of updating it with the new value?
> Two concurrent writers can update the DB in one order and the cache in the other, leaving the cache with the older value. Deleting is safe in either order — the next read loads whatever is in the DB.

> [!question]- 🟡 Q15. Should you delete the cache before or after updating the database?
> After. If you delete first, a reader can miss, read the old value from the DB, and put it back in the cache before your DB write lands. Writing the DB first and then deleting keeps that window tiny.

> [!question]- 🔴 Q16. Even with DB-then-delete, you occasionally see stale values that stick until TTL. How can that happen, and what do you do?
> A reader misses and reads the old value, then the writer updates the DB and deletes the key, then the slow reader writes the old value into the cache. Mitigate with a short TTL, a delayed second delete about a second later, or versioned keys.

---

## 05 · CDN

[Read the notes →](../05-cdn.md)

> [!question]- 🟢 Q17. What is a CDN and what problem does it solve?
> A network of edge servers in many cities that cache content close to users. A user far from the origin gets files in ~20 ms instead of ~250 ms, and most requests never reach our servers, saving load and bandwidth.

> [!question]- 🟡 Q18. Push CDN vs pull CDN — which would you choose?
> Pull by default: the edge fetches from the origin on the first miss and caches it, with no extra work. Push when you know large content will be hot, like a video launch, so you upload it to the edges ahead of time.

> [!question]- 🟡 Q19. You deploy a new version of your JS bundle, but users keep getting the old one. How do you avoid this?
> Use versioned file names with a content hash, like `app.3f9a.js`, and cache them for a long time. Each deploy produces a new URL, so the CDN and browsers fetch it fresh, with no purge needed. Keep the HTML that references the bundle on a short TTL.

> [!question]- 🔴 Q20. After adding a CDN, a user reports seeing another user's account page. What happened?
> A personalized page was cached on the shared CDN under a URL that doesn't include the user. The fix is to mark private responses with `Cache-Control: private` or `no-store`, and only cache public content at the edge.

---

## 06 · Distributed Cache

[Read the notes →](../06-distributed-cache.md)

> [!question]- 🟢 Q21. Why use a distributed cache instead of an in-memory cache on each app server?
> With 50 servers each caching locally, data is duplicated 50 times, invalidating means telling every server, and each server's hit ratio is low. A shared distributed cache keeps one copy per key, gives one consistent view, and scales beyond one machine's RAM.

> [!question]- 🟡 Q22. Redis or Memcached — how do you choose?
> Redis for almost everything: rich data types like sorted sets and hashes, replication, persistence, pub/sub and atomic operations. Memcached when you only need a very simple, multi-threaded key-value blob cache.

> [!question]- 🟡 Q23. How do you shard keys across cache nodes, and why not just use hash mod N?
> With hash mod N, changing N moves almost every key, so adding one node causes a huge miss storm. Consistent hashing moves only about 1/N of keys when a node is added or removed. Redis Cluster does something similar with 16,384 hash slots.

> [!question]- 🔴 Q24. One Redis shard is at 100% CPU while the others are at 20%. What's going on and how do you fix it?
> Most likely a hot key — like a celebrity's post — that lives on that one shard. Fixes: cache it locally on each app server with a short TTL, split it into several copies on different shards and read a random one, or serve reads from that shard's replicas. Also check for very big keys.

---

## 07 · Cache Stampede

[Read the notes →](../07-cache-stampede.md)

> [!question]- 🟡 Q25. What is a cache stampede?
> When a hot key expires or the cache is empty, thousands of requests miss at the same moment and all run the same expensive DB query. The DB slows down, more requests miss, and it can fall over.

> [!question]- 🔴 Q26. Every hour on the hour, your DB CPU spikes to 100% for a few seconds. What's the likely cause and fix?
> Many hot keys were written together with the same 1-hour TTL, so they all expire at once and stampede the DB. Add random jitter to TTLs, let only one request rebuild each key with a lock, and serve the stale value while it refreshes.

---

## 08 · Cache Warming

[Read the notes →](../08-cache-warming.md)

> [!question]- 🟢 Q27. What is cache warming and when do you need it?
> Pre-loading hot keys into the cache before traffic arrives. You need it after deploying a new cache cluster, restarting nodes, or before a big known event, so the database doesn't take 100% of reads from a cold cache.
