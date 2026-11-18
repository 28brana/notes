# Q&A · DBMS — Transactions & Internals

[← Back to DBMS](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 13 · Transactions & ACID

[Read the notes →](../13-transactions-and-acid.md)

> [!question]- 🟢 Q1. What is a transaction and what does ACID stand for?
> A transaction is a group of operations that runs as one unit — it either fully commits or fully rolls back. ACID: Atomicity (all or nothing), Consistency (constraints stay true), Isolation (concurrent transactions don't see each other's half-done work), Durability (committed data survives a crash).

> [!question]- 🟢 Q2. Which DB mechanism gives each ACID property?
> Atomicity and durability come from the write-ahead log — undo for rollback, redo for crash recovery. Isolation comes from locking or MVCC. Consistency comes from constraints like foreign keys and checks, plus correct application logic.

> [!question]- 🟡 Q3. Walk me through the states of a transaction.
> It starts Active. After its last statement it's Partially committed — changes may still be only in memory. Once the commit record is flushed to disk it's Committed. If anything fails it goes to Failed, then Aborted after its changes are undone; the DB may restart it or kill it.

> [!question]- 🔴 Q4. Our checkout calls the payment gateway inside a DB transaction and the DB shows lots of lock waits under load. Why, and what would you change?
> The transaction holds row locks for the whole duration of the slow network call, maybe 1–2 seconds, so every other checkout on those rows waits. I'd move the external call outside the transaction: reserve the order in one short transaction, call the gateway, then confirm in a second short transaction, with idempotency keys and a cleanup job for stuck reservations.

---

## 14 · Schedules & Serializability

[Read the notes →](../14-schedules-and-serializability.md)

> [!question]- 🟢 Q5. What's the difference between a serial and a serializable schedule?
> A serial schedule runs transactions one after another, never interleaved. A serializable schedule is interleaved but produces the same result as some serial order. Serializable gives correctness with concurrency.

> [!question]- 🟡 Q6. How do you check if a schedule is conflict serializable?
> Build a precedence graph: one node per transaction, and an edge Ti → Tj whenever Ti does an operation that conflicts with a later one in Tj — same item, different transactions, at least one write. If the graph has no cycle, the schedule is conflict serializable, and a topological sort gives the equivalent serial order.

> [!question]- 🔴 Q7. Conflict vs view serializability — which one do databases use and why?
> View serializability is broader — it also accepts some schedules with blind writes — but testing it is NP-complete. Conflict serializability is a bit stricter but checkable in polynomial time with a precedence graph, and protocols like 2PL guarantee it. So real systems aim for conflict serializability.

---

## 15 · Locking & 2PL

[Read the notes →](../15-locking-and-2pl.md)

> [!question]- 🟢 Q8. What are shared and exclusive locks?
> A shared lock is for reading — many transactions can hold it on the same row at once. An exclusive lock is for writing — only one transaction can hold it, and it blocks both readers and writers. Only shared plus shared is compatible.

> [!question]- 🟡 Q9. What is two-phase locking and why does it work?
> A transaction first only acquires locks (growing phase), and once it releases any lock it can't acquire new ones (shrinking phase). This means conflicting transactions are ordered by their lock points, which guarantees conflict serializability.

> [!question]- 🟡 Q10. Why do databases use strict 2PL instead of basic 2PL?
> Basic 2PL can release a write lock before commit, so another transaction reads that data, and if the first one aborts the second must abort too — cascading rollback. Strict 2PL holds exclusive locks until commit or abort, so nobody reads uncommitted data and schedules are recoverable and cascadeless.

> [!question]- 🔴 Q11. Two users booked the last seat on a flight. How do you fix it?
> It's a lost update: both read seats_left = 1 and both wrote 0. Options: an atomic conditional update — `UPDATE flights SET seats_left = seats_left - 1 WHERE id = 7 AND seats_left > 0` and check rows affected; a pessimistic `SELECT … FOR UPDATE` lock; or optimistic locking with a version column and a retry. For a hot single row I'd use the atomic conditional update.

---

## 16 · Isolation Levels & MVCC

[Read the notes →](../16-isolation-levels-and-mvcc.md)

> [!question]- 🟢 Q12. Name the four isolation levels and what each prevents.
> Read Uncommitted prevents almost nothing. Read Committed prevents dirty reads. Repeatable Read also prevents non-repeatable reads. Serializable prevents all anomalies including phantoms and write skew. Postgres defaults to Read Committed, MySQL InnoDB to Repeatable Read.

> [!question]- 🟡 Q13. How does MVCC work in Postgres?
> Every row version has xmin (the txn that created it) and xmax (the txn that deleted or replaced it). An update writes a new version instead of overwriting. Each transaction reads with a snapshot and only sees versions committed in that snapshot, so readers never block writers. VACUUM later removes versions nobody can see.

> [!question]- 🟡 Q14. What's write skew and which isolation level allows it?
> Two transactions read the same data, then each updates a different row based on what they read, and together they break a rule — like two on-call doctors both removing themselves. There's no write-write conflict, so snapshot isolation / Repeatable Read allows it. Fix with Serializable or `SELECT … FOR UPDATE` on the rows you checked.

> [!question]- 🔴 Q15. A Postgres table keeps growing on disk even though row count is stable, and queries are slowing down. What's going on?
> Likely MVCC bloat: every update leaves a dead row version, and VACUUM isn't removing them. Common cause is a long-running or idle-in-transaction session holding an old snapshot, which stops cleanup. I'd check `pg_stat_activity` for old transactions, kill them, tune autovacuum for that table, and add idle-transaction timeouts.

---

## 17 · Deadlocks

[Read the notes →](../17-deadlocks.md)

> [!question]- 🟢 Q16. What is a deadlock and what conditions cause it?
> A deadlock is a cycle where each transaction waits for a lock held by the next, so none can continue. It needs four conditions at once: mutual exclusion, hold and wait, no preemption, and circular wait. Breaking any one prevents it.

> [!question]- 🟡 Q17. How do databases detect deadlocks?
> They maintain a wait-for graph with an edge Ti → Tj when Ti waits for a lock Tj holds. A cycle means deadlock. The DB picks a victim — usually the youngest or cheapest — rolls it back and returns an error, and the application retries.

> [!question]- 🟡 Q18. Explain wait-die vs wound-wait.
> Both use start timestamps, older means higher priority. In wait-die, an older txn waits for a younger one, but a younger txn requesting an older one's lock dies. In wound-wait, an older txn wounds (aborts) the younger lock holder, while a younger txn waits. In both, the older one is never aborted, and restarted txns keep their timestamp so they don't starve.

> [!question]- 🔴 Q19. Your money-transfer API throws occasional deadlock errors under load. How do you fix it?
> Transfers A→B and B→A lock the two accounts in opposite orders. I'd always lock accounts in a fixed order, like ascending ID, using `SELECT … FOR UPDATE` on both upfront. Then keep the transaction short and add a retry with small backoff for the rare deadlock error that still happens.

---

## 18 · Recovery & WAL

[Read the notes →](../18-recovery-and-wal.md)

> [!question]- 🟢 Q20. What is write-ahead logging?
> Before any changed data page is written to disk, the change is first recorded in an append-only log on disk. On commit the DB only needs to flush the log, which is a fast sequential write. Data pages are written lazily later.

> [!question]- 🟡 Q21. What do undo and redo do during crash recovery?
> Redo re-applies changes from committed transactions whose pages hadn't reached disk — that gives durability. Undo rolls back changes from transactions that hadn't committed — that gives atomicity. Recovery runs analysis, then redo, then undo.

> [!question]- 🔴 Q22. Why do we need checkpoints, and what's the trade-off in how often they run?
> Without checkpoints, recovery would have to replay the log from the very beginning. A checkpoint flushes dirty pages and marks a point in the log where everything before is safely on disk, so recovery starts there. Frequent checkpoints mean faster recovery but more I/O spikes during normal running.

---

## 19 · Indexing & B+ Trees

[Read the notes →](../19-indexing-b-plus-trees.md)

> [!question]- 🟢 Q23. Clustered vs non-clustered index?
> A clustered index stores the table rows themselves in index order, so there can be only one per table — InnoDB's primary key is an example. A non-clustered index is a separate structure with the key and a pointer to the row (or the primary key), and a table can have many.

> [!question]- 🟡 Q24. Why do databases use B+ trees instead of B trees?
> In a B+ tree, internal nodes hold only keys, so more keys fit per page and the tree is shorter. All data is in the leaves, and the leaves are linked, so range scans just walk right along the leaves. Lookups are predictable — always root to leaf.

> [!question]- 🟡 Q25. When would you use a hash index over a B+ tree?
> Only for pure equality lookups where you never need ranges or sorting — hashing gives O(1) for `=`. It can't do `>`, `BETWEEN`, `ORDER BY` or prefix `LIKE`. That's why B+ trees are the default for almost everything.

> [!question]- 🔴 Q26. Inserts into an InnoDB table with a UUID primary key got slow as the table grew. Why?
> InnoDB clusters the table on the primary key. Random UUIDv4 values land on random pages, causing page splits, poor cache hits and fragmentation; and every secondary index stores that 16-byte PK too. Switching to an auto-increment or time-ordered ID like UUIDv7 makes inserts append at the right edge of the tree.

---

## 20 · Storage & File Organization

[Read the notes →](../20-storage-and-file-organization.md)

> [!question]- 🟢 Q27. What is a page and a buffer pool?
> A page is the fixed-size block (about 8–16 KB) the database reads and writes — it never reads a single row from disk. The buffer pool is the RAM cache of pages; hits are fast, misses go to disk, and modified pages are flushed later.

> [!question]- 🟡 Q28. Heap file vs clustered organization — Postgres vs MySQL?
> Postgres stores tables as heaps — rows go wherever there's space, and all indexes point to the row's physical location. MySQL InnoDB stores rows inside the primary key B+ tree, so PK lookups and ranges are fast, but secondary indexes need a second lookup via the PK.

---

## 21 · Query Processing

[Read the notes →](../21-query-processing.md)

> [!question]- 🟢 Q29. What happens inside the DB when you run a query?
> The parser checks syntax and resolves names into a tree, the rewriter expands views and simplifies, the cost-based optimizer uses statistics to pick the cheapest plan, and the executor runs that plan and returns rows.

> [!question]- 🟡 Q30. Compare nested loop, hash join and merge join.
> Nested loop scans the outer side and looks up matches in the inner — best when the outer is small and the inner has an index. Hash join builds a hash table on the smaller side and probes it with the bigger — best for large equality joins. Merge join walks two sorted inputs together — best when both are already sorted by the key.

> [!question]- 🟡 Q31. Why might the optimizer choose a bad plan?
> Usually stale or missing statistics, so its row estimates are way off — for example it expects 10 rows and gets 1 million, then picks a nested loop. Also correlated columns it treats as independent, or functions hiding the column from an index. Running `ANALYZE` and comparing estimated vs actual rows in `EXPLAIN ANALYZE` reveals this.

> [!question]- 🔴 Q32. A report query was fast last month but now takes minutes after a big data import. What do you check?
> I'd run `EXPLAIN ANALYZE` and compare it with the old plan. After a bulk import, statistics are often stale, so the planner may switch to a nested loop or skip an index; running `ANALYZE` usually fixes it. I'd also check if a hash join now spills to disk because work_mem is too small, and whether new data needs a different index.

---

## 22 · NoSQL Basics

[Read the notes →](../22-nosql-basics.md)

> [!question]- 🟢 Q33. What are the main types of NoSQL databases?
> Key-value like Redis for caches and sessions, document like MongoDB for flexible JSON records, wide-column like Cassandra for huge write volumes, and graph like Neo4j for relationship-heavy queries.

> [!question]- 🟡 Q34. What is BASE and how is it different from ACID?
> BASE means basically available, soft state, eventually consistent. ACID prioritizes correctness with strong, immediate consistency; BASE prioritizes availability and scale, letting replicas catch up after a short delay.

> [!question]- 🔴 Q35. You're storing chat messages for 50M users. Postgres or Cassandra, and how would you model it?
> If writes are hundreds of thousands per second and reads are always "latest messages for a chat", Cassandra fits: partition key chat_id (maybe plus a time bucket so partitions don't grow forever), clustering key message time descending. Users, payments and other relational data would stay in Postgres.

---

## 23 · SQL Interview Problems

[Read the notes →](../23-sql-interview-problems.md)

> [!question]- 🟢 Q36. Find emails that appear more than once in the users table.
> Group by the column and filter groups with HAVING:
> ```sql
> SELECT email, COUNT(*) FROM users GROUP BY email HAVING COUNT(*) > 1;
> ```

> [!question]- 🟡 Q37. Find the second highest salary in each department.
> Rank within each department with DENSE_RANK so ties share a rank, then keep rank 2:
> ```sql
> SELECT dept_id, name, salary FROM (
>   SELECT *, DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk
>   FROM employees) t
> WHERE rnk = 2;
> ```

> [!question]- 🟡 Q38. Find employees who earn more than their manager.
> Self join the table — one alias for the employee, one for the manager:
> ```sql
> SELECT e.name FROM employees e
> JOIN employees m ON e.manager_id = m.id
> WHERE e.salary > m.salary;
> ```

> [!question]- 🔴 Q39. Find users who logged in on 3 or more consecutive days.
> It's gaps and islands: for consecutive dates, date minus ROW_NUMBER stays constant, so group by that. Use distinct dates first so two logins on one day don't break it, then keep groups with `COUNT(*) >= 3`. In Postgres: `login_date - CAST(ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) AS int)` as the group key.
