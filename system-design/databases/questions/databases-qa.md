# Q&A · Databases

[← Back to Databases](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 01 · Database Types

[Read the notes →](../01-database-types.md)

> [!question]- 🟢 Q1. Name the main types of databases and one use case for each.
> Relational for users and payments, document for profiles and catalogs, key-value for sessions and cache, wide-column for huge write streams like chat messages, search engines for text search, time-series for metrics, graph for multi-hop relationships, and vector for similarity search.

> [!question]- 🟡 Q2. How do you decide which database to use?
> I look at the data shape, how it's read (by key, by range, by text, by similarity), the read/write ratio, and how strict consistency must be. Then I pick the type built for that access pattern. If nothing special is needed, I default to Postgres.

> [!question]- 🔴 Q3. Your design uses Postgres, Redis, Elasticsearch, and Cassandra. The interviewer asks "isn't that too many?" How do you respond?
> Each one serves a specific access pattern, but every store adds ops cost and sync complexity. For an MVP I'd run everything on Postgres, and only add the others when a measured need appears. Postgres stays the source of truth, and the others are fed by CDC so they can be rebuilt.

---

## 02 · SQL vs NoSQL

[Read the notes →](../02-sql-vs-nosql.md)

> [!question]- 🟢 Q4. What are the main differences between SQL and NoSQL databases?
> SQL has a fixed schema, joins, and full ACID transactions, and usually scales vertically first. NoSQL has flexible schemas, usually no joins, often weaker or tunable consistency, and is built to shard horizontally. NoSQL is fast for queries you designed for, but poor at ad-hoc ones.

> [!question]- 🟡 Q5. What does "model your queries first" mean in NoSQL?
> In databases like Cassandra or DynamoDB, you design tables around the exact queries you'll run, often one table per query, and duplicate data between them. In SQL you normalize the data and write any query later. Changing queries in NoSQL often means new tables and backfills.

> [!question]- 🟡 Q6. Is it true that SQL databases can't scale?
> No. With indexes, read replicas, caching, partitioning, and eventually sharding, SQL handles very large apps. What's harder is scaling writes past one primary, which needs sharding. NewSQL like Spanner or CockroachDB gives SQL with built-in horizontal scale.

> [!question]- 🔴 Q7. You're designing a chat app with billions of messages a day. Which database for messages, and which for user accounts?
> Messages in Cassandra, partitioned by conversation ID and clustered by timestamp, because it's append-heavy and always read as "latest N in this chat". User accounts in Postgres because they're relational, smaller, and need transactions and unique constraints like email.

---

## 03 · ACID Transactions

[Read the notes →](../03-acid-transactions.md)

> [!question]- 🟢 Q8. What does ACID stand for?
> Atomicity: all or nothing. Consistency: constraints stay true. Isolation: concurrent transactions don't interfere, as if they ran one by one to some degree. Durability: once committed, the data survives crashes. It's built on a write-ahead log plus locks or MVCC.

> [!question]- 🟡 Q9. Explain dirty read, non-repeatable read, and phantom read.
> A dirty read is seeing another transaction's uncommitted change. A non-repeatable read is reading the same row twice and getting different values because someone committed in between. A phantom is re-running a range query and seeing new rows appear.

> [!question]- 🟡 Q10. What isolation level do most databases use by default, and what does it allow?
> Usually Read Committed — Postgres, Oracle, and SQL Server default to it, while MySQL defaults to Repeatable Read. Read Committed stops dirty reads but still allows non-repeatable reads, phantoms, and lost updates. Serializable stops everything but costs throughput.

> [!question]- 🔴 Q11. Two users like a post at the same time and the count goes up by only 1. Why, and how do you fix it?
> That's a lost update: both read 100, both write 101. Read Committed doesn't prevent it. Fix it with an atomic `UPDATE ... SET likes = likes + 1`, or `SELECT ... FOR UPDATE`, or optimistic locking with a version column and retry.

---

## 04 · B-Trees

[Read the notes →](../04-b-trees.md)

> [!question]- 🟢 Q12. What is a B-tree and why do databases use it?
> It's a balanced, sorted tree where each node is a disk page holding hundreds of keys. Because of the high branching factor, billions of rows are only 3–4 levels deep, so lookups need very few disk reads. It also supports range scans because keys are sorted.

> [!question]- 🟡 Q13. Why is a B-tree slower for writes than an LSM tree?
> B-trees update pages in place, so writes are random disk I/O, and changing a small row rewrites a whole page plus the WAL. Page splits add more work. LSM trees turn writes into sequential appends instead.

> [!question]- 🔴 Q14. Inserts into a big table got slower over time after you switched to random UUID primary keys. Why?
> Random UUIDs land on random leaf pages, so each insert touches a different page, causing cache misses, page splits, and fragmentation. Time-ordered IDs like UUIDv7 or Snowflake IDs append to the right edge of the B-tree, keeping inserts cheap.

---

## 05 · LSM Trees

[Read the notes →](../05-lsm-trees.md)

> [!question]- 🟢 Q15. How does an LSM tree handle a write?
> It appends to a write-ahead log for durability and inserts into a sorted in-memory memtable. When the memtable is full, it's flushed to disk as an immutable sorted file called an SSTable. Background compaction later merges SSTables.

> [!question]- 🟡 Q16. How do LSM trees keep reads fast if data is spread across many files?
> Each SSTable has a bloom filter that quickly says "this key is definitely not here", so most files are skipped. A sparse index inside each file jumps close to the key. Compaction also keeps the number of files low.

> [!question]- 🔴 Q17. A Cassandra cluster's read latency keeps rising after a big bulk delete. What's going on?
> Deletes are written as tombstones, not removed right away. Reads have to scan past all those tombstones until compaction cleans them up. Fixes are tuning compaction, avoiding mass deletes (use TTLs or drop whole partitions or tables), and checking that compaction keeps up.

---

## 06 · Relational Databases

[Read the notes →](../06-relational-databases.md)

> [!question]- 🟢 Q18. What are primary keys and foreign keys?
> A primary key uniquely identifies a row in a table. A foreign key is a column that points to another table's primary key, and the database enforces that it points to a real row, so you can't have a comment on a deleted post.

> [!question]- 🟡 Q19. What is normalization, and when would you denormalize?
> Normalization stores each fact once to avoid conflicting copies and make updates simple. I'd denormalize a specific hot read path, like storing like_count on the post, once I've measured that joins are too slow — accepting the work of keeping copies in sync.

> [!question]- 🟡 Q20. What is the N+1 query problem?
> You load a list of 50 posts with one query, then run one query per post to get its author — 51 queries. It's often hidden by ORMs. Fix it with a join or a single batched `WHERE id IN (...)` query.

> [!question]- 🔴 Q21. Your Postgres primary is at 90% CPU. What do you try, in order, before sharding?
> First find slow queries and add the right indexes. Then add caching for hot reads and move read traffic to read replicas. Add connection pooling, and scale up the machine. Partition big tables. Only if writes still exceed one primary would I shard.

---

## 07 · Document Databases

[Read the notes →](../07-document-databases.md)

> [!question]- 🟢 Q22. What is a document database and when would you use it?
> It stores each record as a JSON-like document with a flexible schema, so data read together is stored together. It's good for self-contained data like profiles, product catalogs, and CMS content, where one read gets everything without joins.

> [!question]- 🟡 Q23. When do you embed data in a document versus reference it?
> Embed when the data belongs to the parent and is read with it, like settings or an address. Reference when it's shared by many documents or can grow without limit, like a post's comments, because documents have size limits and get slow to rewrite.

---

## 08 · Key-Value Stores

[Read the notes →](../08-key-value-stores.md)

> [!question]- 🟢 Q24. What is a key-value store and what is it good for?
> It's a distributed hash map: get, put, and delete by key in O(1). It's great for sessions, caching, counters, rate limits, and any data you always access by ID. Redis is the in-memory example; DynamoDB is the durable, managed one.

> [!question]- 🟡 Q25. Redis vs DynamoDB — when do you use which?
> Redis is in-memory and sub-millisecond, limited by RAM, and durability is optional — ideal for caching, sessions, and leaderboards. DynamoDB is durable, replicated, and practically unlimited in size with single-digit ms latency — ideal as a primary store for key-based access at huge scale.

> [!question]- 🟡 Q26. Why does the choice of partition key matter so much in DynamoDB?
> The partition key decides which node stores the item. A key with few distinct values, or one that's very popular, sends most traffic to one partition and causes throttling. A high-cardinality key like user ID spreads load evenly.

> [!question]- 🔴 Q27. A celebrity posts and one Redis node hits 100% CPU while others are idle. What happened and how do you fix it?
> It's a hot key — all reads for that post hash to the same node. I'd add a small local in-process cache on app servers for very hot keys, or replicate the key as `key#1…key#N` across nodes and read a random copy. Read replicas for that shard also help.

---

## 09 · Full-Text Search

[Read the notes →](../09-full-text-search.md)

> [!question]- 🟢 Q28. What is an inverted index?
> It maps each word to the list of documents that contain it, like the index at the back of a book. To search, you look up each query word and intersect the lists, so there's no need to scan every document.

> [!question]- 🟡 Q29. Why not just use SQL LIKE '%word%' for search?
> A leading wildcard can't use a normal B-tree index, so it scans the whole table. It also can't handle plurals, typos, or relevance ranking. A search engine handles all of that with analysis, an inverted index, and BM25 scoring.

> [!question]- 🔴 Q30. A user creates a post but can't find it in search for a few seconds. Is this a bug?
> Probably not. Elasticsearch is a derived copy synced from the main DB through CDC or events, and it also refreshes its index about once a second. So search is eventually consistent. If the user must see their own post instantly, show it from the main DB on their own profile or feed.

---

## 10 · Wide-Column Databases

[Read the notes →](../10-wide-column-databases.md)

> [!question]- 🟢 Q31. What are the partition key and clustering key in Cassandra?
> The partition key is hashed to decide which nodes store the data, and groups related rows together. The clustering key sorts rows inside the partition on disk, so range queries like "latest 50 messages" are one sequential read.

> [!question]- 🟡 Q32. How does Cassandra give you consistent reads if it's leaderless?
> Through tunable consistency. With 3 replicas, writing to a quorum of 2 and reading from a quorum of 2 guarantees the read set overlaps the write set (R + W > N), so you see the latest write. Using ONE is faster but may return stale data.

> [!question]- 🔴 Q33. Writes in your Cassandra cluster all hit one node. Your partition key is the date. What's wrong?
> All of today's writes share one partition key, so they go to the same replica set — a hot partition. Use a high-cardinality key like user or device ID, or add a bucket to spread writes, like (date, bucket 0–99).

---

## 11 · Time-Series Databases

[Read the notes →](../11-time-series-databases.md)

> [!question]- 🟢 Q34. What makes a time-series database different from a normal database?
> It's built for append-only timestamped data read in time ranges. It partitions data into time chunks, compresses heavily, and has built-in downsampling and retention, so it stores metrics far smaller and queries them far faster.

> [!question]- 🔴 Q35. Your metrics DB's memory usage exploded after someone added a user_id tag. Why?
> Each unique combination of tags is a separate series. Adding user_id with millions of values turned a few thousand series into millions — high cardinality. Remove it as a tag; put per-user data in logs or an analytics store instead.

---

## 12 · Graph Databases

[Read the notes →](../12-graph-databases.md)

> [!question]- 🟢 Q36. When would you choose a graph database over SQL?
> When the queries follow relationships several hops deep, like friends-of-friends, fraud rings, or recommendations. A graph DB stores direct pointers between nodes, so each hop is cheap, while SQL needs a join per hop over huge tables.

> [!question]- 🟡 Q37. Should a social network store its follow graph in a graph database?
> Not necessarily. Most social queries are one hop — "who do I follow" — which a sharded SQL or key-value store with caching handles well; Facebook's TAO is built on MySQL. A graph DB helps for deep multi-hop features, and graphs are hard to shard at that scale.

---

## 13 · Vector Databases

[Read the notes →](../13-vector-databases.md)

> [!question]- 🟢 Q38. What is an embedding and what does a vector database do with it?
> An embedding is a list of numbers produced by a model that captures meaning — similar things get nearby vectors. A vector database stores these and finds the nearest ones to a query vector, enabling semantic search, recommendations, and RAG.

> [!question]- 🟡 Q39. What is approximate nearest neighbor search, and how does HNSW work at a high level?
> Exact search compares against every vector, which is too slow at scale. ANN gives up a little recall for big speed. HNSW builds a layered graph — sparse long-range links on top, dense local links at the bottom — and greedily walks down the layers toward the query's nearest neighbours.

> [!question]- 🔴 Q40. You upgraded to a better embedding model and search quality dropped badly. Why?
> The stored vectors were made with the old model and queries now use the new one — the two spaces aren't compatible, so distances are meaningless. You must re-embed all documents with the new model and rebuild the index, ideally in a parallel index before switching.
