# Q&A · Data Structures for Scale

[← Back to Data Structures for Scale](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 01 · Geohash

[Read the notes →](../01-geohash.md)

> [!question]- 🟢 Q1. What is a geohash and why is it useful?
> A geohash turns a latitude/longitude into a short string by repeatedly halving the map and interleaving lat and lng bits. Places with the same prefix are in the same cell, so "find nearby" becomes a prefix query on a normal index like a B-tree or Redis.

> [!question]- 🟡 Q2. How do you pick the geohash length for a search?
> Match the cell size to the search radius. Each extra character makes cells ~32× smaller: 5 chars is about 5 km, 6 chars about 1 km, 7 chars about 150 m. For "drivers within 1 km" I'd use 6 characters plus the neighbouring cells.

> [!question]- 🟡 Q3. Two users are 5 metres apart but have completely different geohashes. Why, and how do you handle it?
> They're on opposite sides of a cell border, so their prefixes differ even though they're close. The fix is to always search the user's cell plus its 8 neighbours, then filter candidates by exact distance.

> [!question]- 🔴 Q4. Your "nearby drivers" service using geohash is slow in Manhattan but fine elsewhere. What's going on?
> Geohash uses a fixed grid, so one cell in a dense city holds tens of thousands of drivers and each query returns too much. Options: use a longer geohash in dense areas, switch to an adaptive structure like a quadtree, or use H3 cells with a smaller resolution in cities. Also cap results and sort by distance.

---

## 02 · Quad Trees

[Read the notes →](../02-quad-trees.md)

> [!question]- 🟢 Q5. What is a quadtree?
> A tree where each node is a box on the map with four children: NW, NE, SW, SE. A box splits only when it holds more than a set number of points, say 500, so dense areas get small cells and empty areas stay big.

> [!question]- 🟡 Q6. Quadtree vs geohash — when would you choose a quadtree?
> When density is very uneven and you want every cell to hold a similar number of points, like a Yelp-style restaurant search. Geohash is simpler and works in any DB, but its fixed grid gives huge cells in cities and empty cells elsewhere.

> [!question]- 🔴 Q7. Would you use a quadtree to track 1 million Uber drivers updating every 4 seconds?
> Probably not as the main structure. That's 250K updates per second, each moving a point between leaves and possibly splitting or merging nodes, which needs locking. I'd map drivers to geohash or H3 cells in Redis instead and keep quadtrees for mostly static data like restaurants.

---

## 03 · R-Trees

[Read the notes →](../03-r-trees.md)

> [!question]- 🟢 Q8. What problem does an R-tree solve that a quadtree doesn't handle well?
> Indexing shapes, not just points — roads, parks, delivery zones. Each object gets a bounding rectangle, and rectangles are nested into a balanced tree, so a query only explores boxes that overlap the area it cares about.

> [!question]- 🟡 Q9. You need "which delivery zone contains this address?" for 10,000 polygon zones. How?
> Store the zones as polygons in Postgres with PostGIS and add a spatial (GiST, R-tree style) index. The index finds the few zones whose bounding boxes contain the point, then an exact point-in-polygon check picks the right one. No need to build anything custom.

---

## 04 · S2 and H3

[Read the notes →](../04-s2-and-h3.md)

> [!question]- 🟢 Q10. What are S2 and H3?
> Libraries that divide the whole globe into hierarchical cells, each with a 64-bit ID. S2 from Google projects the Earth onto a cube and uses squares; H3 from Uber uses hexagons. Both give consistent cell sizes worldwide, unlike geohash.

> [!question]- 🟡 Q11. Why did Uber choose hexagons?
> All 6 neighbours of a hexagon are the same distance from its centre, so "search in rings around me" and demand heatmaps are smooth and fair. With squares, diagonal neighbours are farther than side neighbours, which skews distance-based logic like surge pricing.

> [!question]- 🔴 Q12. Your geohash-based app is expanding to Norway and Iceland, and nearby search behaves oddly there. Why, and what would you change?
> Geohash treats the Earth as a flat rectangle, so cells get much narrower near the poles and a fixed-length hash no longer matches the same radius. I'd move to S2 or H3 cell IDs, which keep cell sizes roughly equal across the globe, and search a ring of neighbouring cells.

---

## 05 · Bloom Filters

[Read the notes →](../05-bloom-filters.md)

> [!question]- 🟢 Q13. What is a Bloom filter?
> A bit array plus a few hash functions that answers "is X in the set?" Adding X sets k bits; checking X looks at those k bits. If any is 0, X is definitely not there; if all are 1, X is probably there.

> [!question]- 🟢 Q14. Can a Bloom filter give false negatives? False positives?
> Never false negatives — once bits are set, they stay set, so an added item always passes. False positives yes — other items may have set the same bits. That's why "maybe" answers go on to a real lookup.

> [!question]- 🟡 Q15. How do Bloom filters speed up Cassandra or RocksDB reads?
> These LSM databases store data in many on-disk files. Each file has a Bloom filter in memory, so a read skips every file that definitely doesn't hold the key. Most reads touch one or two files on disk instead of all of them.

> [!question]- 🔴 Q16. You built a "username taken" Bloom filter. Now users can delete accounts, and freed names still show as taken. Also, after 2 years, more names show as taken wrongly. What happened?
> First, a plain Bloom filter can't delete, so freed names keep their bits set — use a Cuckoo filter or rebuild periodically from the DB. Second, it was sized for fewer users; as it filled, more bits became 1 and the false positive rate climbed. Resize based on growth, and always confirm "maybe" with the database.

---

## 06 · Cuckoo Filter

[Read the notes →](../06-cuckoo-filter.md)

> [!question]- 🟢 Q17. What does a Cuckoo filter add over a Bloom filter?
> Deletes. It stores small fingerprints in one of two candidate buckets, so you can find and remove one. It's also slightly more space-efficient at low false positive rates.

> [!question]- 🟡 Q18. What are the downsides of a Cuckoo filter?
> Inserts can fail when the table is around 95% full, because the evict-and-move chain gets too long. And deleting an item that was never added can remove someone else's matching fingerprint, creating a false negative.

---

## 07 · Skip Lists

[Read the notes →](../07-skip-lists.md)

> [!question]- 🟢 Q19. What is a skip list?
> A sorted linked list with extra "express lane" levels on top, built by random coin flips. Search starts at the top level and drops down, giving O(log n) average search, insert, and delete.

> [!question]- 🟡 Q20. Why does Redis use a skip list for sorted sets instead of a balanced tree?
> It's simpler to implement, range queries are just a walk along the bottom list, and it performs as well as a tree on average. Redis pairs it with a hash map so looking up a member's score is O(1).

> [!question]- 🔴 Q21. Design a real-time leaderboard for a game with 50 million players.
> Use a Redis sorted set: ZINCRBY to update scores, ZREVRANGE for the top 100, ZREVRANK for "my rank" — all O(log n) thanks to the skip list. At 50M members one set is big, so I'd shard by region or league and keep a small global top-N set. For exact global rank across shards, approximate with score buckets.

---

## 08 · Count-Min Sketch

[Read the notes →](../08-count-min-sketch.md)

> [!question]- 🟢 Q22. What is a Count-Min Sketch?
> A small grid of counters with one hash function per row. Each item increments one counter per row; its estimated count is the minimum of those counters. It can overcount because of collisions, but never undercounts.

> [!question]- 🔴 Q23. How would you find the top 10 trending hashtags at 50,000 posts per second?
> Feed every hashtag into a Count-Min Sketch per minute and keep a min-heap of the top 10 by estimated count. Merge the last 60 minute-sketches for an hourly window. Memory stays a few MB regardless of how many distinct hashtags exist, and exact counts can be recomputed offline if needed.

---

## 09 · HyperLogLog

[Read the notes →](../09-hyperloglog.md)

> [!question]- 🟢 Q24. What is HyperLogLog used for?
> Counting distinct items approximately, like unique visitors. It hashes items, tracks the longest run of leading zeros per bucket, and estimates the count from that. In Redis it's PFADD/PFCOUNT, about 12 KB per counter with ~0.81% error.

> [!question]- 🟡 Q25. You have daily unique-visitor counts. Can you add them to get weekly uniques?
> No — the same user visiting on multiple days would be counted several times. Keep one HyperLogLog per day and merge them with PFMERGE; merging takes the max per bucket, so repeat users are counted once.

---

## 10 · Merkle Trees

[Read the notes →](../10-merkle-trees.md)

> [!question]- 🟢 Q26. What is a Merkle tree?
> A tree of hashes: leaves are hashes of data blocks, and each parent is the hash of its children, up to a single root. If any data changes, the root changes, so the root acts as a fingerprint of everything.

> [!question]- 🟡 Q27. How do Cassandra or Dynamo use Merkle trees?
> For anti-entropy repair. Two replicas build Merkle trees over their key ranges and compare roots. If they differ, they walk down only the differing branches to find the exact out-of-sync ranges, then stream just those ranges.

> [!question]- 🔴 Q28. Two replicas each have 1 billion rows and you suspect a few hundred differ. Copying everything takes hours. What do you do?
> Build a Merkle tree on each replica over key ranges and compare top-down. Matching subtrees are skipped entirely, so only about log n hash comparisons per differing range are needed. Then sync just those ranges and resolve conflicts with the system's rule, like last-write-wins or vector clocks.

---

## 11 · MinHash

[Read the notes →](../11-minhash.md)

> [!question]- 🟢 Q29. What is MinHash used for?
> Estimating how similar two sets are, usually documents broken into shingles. Each document gets a small signature of minimum hash values, and the fraction of matching values approximates their Jaccard similarity. Classic use: near-duplicate detection.

> [!question]- 🔴 Q30. Spammers repost the same message with small edits and your MD5-based dedup misses them. How do you catch them at scale?
> MD5 changes completely with one character, so it only catches exact copies. I'd shingle each post, compute a MinHash signature, and use LSH banding so similar posts land in the same bucket. Then compare only those candidates and flag anything above, say, 85% similarity.
