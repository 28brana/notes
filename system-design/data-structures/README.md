# System Design · Data Structures for Scale

[← Back to System Design](../README.md) · [Interview Q&A →](questions/data-structures-qa.md) · [Practice questions 📝](questions/practice-questions.md)

Special data structures that make huge problems cheap: "find drivers near me", "is this username taken", "how many unique visitors", "are these two replicas in sync".
Interviewers love them because naming the right one turns a 10 TB problem into a 12 KB one.

| # | Topic | In one line | Priority |
|---|---|---|---|
| 01 | [Geohash](01-geohash.md) | Turn lat/long into a **string prefix** so "nearby" becomes a prefix lookup | 🔥 High |
| 02 | [Quad Trees](02-quad-trees.md) | Split the map into 4 boxes **only where it's crowded** | ⚡ Medium |
| 03 | [R-Trees](03-r-trees.md) | Group **shapes** (not just points) into nested bounding boxes | ⚡ Medium |
| 04 | [S2 and H3](04-s2-and-h3.md) | Google's squares on a sphere and Uber's hexagons — **plus geo comparison table** | ⚡ Medium |
| 05 | [Bloom Filters](05-bloom-filters.md) | "**Definitely not** / **maybe** in the set" with tiny memory | 🔥 High |
| 06 | [Cuckoo Filter](06-cuckoo-filter.md) | A Bloom filter that **supports delete** | 💤 Low |
| 07 | [Skip Lists](07-skip-lists.md) | A sorted linked list with **express lanes** — Redis sorted sets | ⚡ Medium |
| 08 | [Count-Min Sketch](08-count-min-sketch.md) | **Approximate counts** for top-K and trending items | 💤 Low |
| 09 | [HyperLogLog](09-hyperloglog.md) | Count **unique** items: billions in ~12 KB, ~0.81% error | 💤 Low |
| 10 | [Merkle Trees](10-merkle-trees.md) | A **tree of hashes** to find what differs between two copies fast | ⚡ Medium |
| 11 | [MinHash](11-minhash.md) | Estimate how **similar** two sets are — near-duplicate detection | 💤 Low |

> [!example] Cheat sheet — "which structure?"
> | Interview ask | Reach for |
> |---|---|
> | **Find nearby drivers / restaurants** | Geohash, Quadtree, S2 / H3 |
> | **Has this been seen before?** | Bloom filter (Cuckoo if you need deletes) |
> | **Leaderboard / ranking** | Skip list (Redis sorted set) |
> | **Top-K / trending** | Count-Min Sketch + min-heap |
> | **Unique visitor count** | HyperLogLog |
> | **Sync replicas cheaply** | Merkle tree |
> | **Near-duplicate documents** | MinHash + LSH |
