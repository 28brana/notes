# System Design · Databases

[← Back to System Design](../README.md) · [Interview Q&A →](questions/databases-qa.md) · [Practice questions 📝](questions/practice-questions.md)

Almost every design interview has a "which database and why?" moment. This chapter covers the main database types, how they store data on disk, and the guarantees (ACID) they give you.

| # | Topic | In one line | Priority |
|---|---|---|---|
| 01 | [Database Types](01-database-types.md) | A map of the 8 main database families and when to pick each | 🔥 High |
| 02 | [SQL vs NoSQL](02-sql-vs-nosql.md) | Fixed schema + joins + transactions vs flexible schema + easy horizontal scale | 🔥 High |
| 03 | [ACID Transactions](03-acid-transactions.md) | All-or-nothing changes, plus isolation levels and the anomalies they stop | 🔥 High |
| 04 | [B-Trees](04-b-trees.md) | The read-optimized, update-in-place index behind most SQL databases | ⚡ Medium |
| 05 | [LSM Trees](05-lsm-trees.md) | The write-optimized, append-then-compact engine behind Cassandra and RocksDB | ⚡ Medium |
| 06 | [Relational Databases](06-relational-databases.md) | Tables, keys, joins, and normalization — the safe default (Postgres, MySQL) | 🔥 High |
| 07 | [Document Databases](07-document-databases.md) | Store each record as a JSON document (MongoDB) | ⚡ Medium |
| 08 | [Key-Value Stores](08-key-value-stores.md) | Get/put by key — the fastest and simplest model (Redis, DynamoDB) | 🔥 High |
| 09 | [Full-Text Search](09-full-text-search.md) | Inverted index for fast keyword search and ranking (Elasticsearch) | ⚡ Medium |
| 10 | [Wide-Column Databases](10-wide-column-databases.md) | Partition key + sorted rows for huge write-heavy data (Cassandra) | ⚡ Medium |
| 11 | [Time-Series Databases](11-time-series-databases.md) | Optimized for timestamped metrics: fast ingest, rollups, retention (InfluxDB) | ⚡ Medium |
| 12 | [Graph Databases](12-graph-databases.md) | Nodes and edges for relationship-heavy queries (Neo4j) | 💤 Low |
| 13 | [Vector Databases](13-vector-databases.md) | Find "similar" items via embeddings and approximate nearest neighbor search | ⚡ Medium |
