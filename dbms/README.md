# DBMS

[← Back to all subjects](../README.md)

Database fundamentals from the college syllabus plus the SQL that every backend interview tests — keys, joins, normalization, transactions, locking, indexes. For architecture-level choices (SQL vs NoSQL, sharding, replication), see [System Design · Databases](../system-design/databases/README.md) and [Database Scaling](../system-design/database-scaling/README.md).

## Part 1 · SQL & Data Modeling
[Interview Q&A →](questions/sql-and-modeling-qa.md) · [Practice questions 📝](questions/sql-and-modeling-practice.md)

| # | Topic | In one line | Priority |
|---|---|---|---|
| 01 | [What is a DBMS](01-what-is-dbms.md) | Software that stores, queries and protects shared data — beats plain files | 🔥 High |
| 02 | [ER Model](02-er-model.md) | Entities, attributes, relationships and **cardinality** → tables | 🔥 High |
| 03 | [Relational Model & Keys](03-relational-model-and-keys.md) | Super → candidate → primary keys, foreign / composite / surrogate | 🔥 High |
| 04 | [Relational Algebra](04-relational-algebra.md) | σ, π, ⋈ — the math SQL compiles into | 💤 Low |
| 05 | [SQL Basics](05-sql-basics.md) | DDL/DML, **order of execution**, NULL rules, DELETE vs TRUNCATE | 🔥 High |
| 06 | [Joins](06-joins.md) | Inner, left, right, full, self, cross — with result examples | 🔥 High |
| 07 | [Subqueries & CTEs](07-subqueries-and-ctes.md) | Correlated subqueries, `WITH`, recursive CTEs for trees | 🔥 High |
| 08 | [GROUP BY & Aggregates](08-group-by-and-aggregates.md) | One row per group; **WHERE vs HAVING** | 🔥 High |
| 09 | [Window Functions](09-window-functions.md) | ROW_NUMBER vs RANK vs DENSE_RANK, LAG/LEAD, running totals | 🔥 High |
| 10 | [Views, Procedures & Triggers](10-views-procedures-triggers.md) | Saved queries, code inside the DB, auto-run on changes | ⚡ Medium |
| 11 | [Functional Dependencies](11-functional-dependencies.md) | A → B, closures, finding keys | ⚡ Medium |
| 12 | [Normalization](12-normalization.md) | 1NF → 2NF → 3NF → BCNF, and when to denormalize | 🔥 High |

## Part 2 · Transactions & Internals
[Interview Q&A →](questions/transactions-and-internals-qa.md) · [Practice questions 📝](questions/transactions-and-internals-practice.md)

| # | Topic | In one line | Priority |
|---|---|---|---|
| 13 | [Transactions & ACID](13-transactions-and-acid.md) | All-or-nothing units of work: atomicity, consistency, isolation, durability | 🔥 High |
| 14 | [Schedules & Serializability](14-schedules-and-serializability.md) | When interleaved transactions are as safe as running one by one | ⚡ Medium |
| 15 | [Locking & 2PL](15-locking-and-2pl.md) | Shared / exclusive locks and two-phase locking | 🔥 High |
| 16 | [Isolation Levels & MVCC](16-isolation-levels-and-mvcc.md) | Dirty / non-repeatable / phantom reads, and multi-version snapshots | 🔥 High |
| 17 | [Deadlocks](17-deadlocks.md) | Two transactions waiting on each other — detect, prevent, retry | 🔥 High |
| 18 | [Recovery & WAL](18-recovery-and-wal.md) | Write-ahead log, checkpoints, redo / undo after a crash | ⚡ Medium |
| 19 | [Indexing & B+ Trees](19-indexing-b-plus-trees.md) | How indexes find rows fast — clustered, composite, covering | 🔥 High |
| 20 | [Storage & File Organization](20-storage-and-file-organization.md) | Pages, heap files, how rows sit on disk | 💤 Low |
| 21 | [Query Processing](21-query-processing.md) | Parser → optimizer → plan; reading `EXPLAIN`, join algorithms | 🔥 High |
| 22 | [NoSQL Basics](22-nosql-basics.md) | Document, key-value, column, graph — the DBMS view | ⚡ Medium |
| 23 | [SQL Interview Problems](23-sql-interview-problems.md) | Nth highest, top N per group, duplicates, gaps — solved patterns | 🔥 High |
