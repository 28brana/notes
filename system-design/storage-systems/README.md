# System Design · Storage Systems

[← Back to System Design](../README.md) · [Interview Q&A →](questions/storage-systems-qa.md) · [Practice questions 📝](questions/practice-questions.md)

Where the bytes actually live — disks for databases, shared file systems for big data, and object stores for photos and backups. Interviewers expect you to pick the right one and explain how it stays durable.

| # | Topic | In one line | Priority |
|---|---|---|---|
| 01 | [Block Storage](01-block-storage.md) | Raw virtual disk attached to one server — fast, for databases. Includes block vs file vs object table | ⚡ Medium |
| 02 | [Distributed File Systems](02-distributed-file-systems.md) | HDFS/GFS — big chunks, 3 replicas, a metadata NameNode | 💤 Low |
| 03 | [Object Storage](03-object-storage.md) | S3 buckets and keys, presigned uploads, lifecycle tiers | ⚡ Medium |
| 04 | [Erasure Coding](04-erasure-coding.md) | Data + parity pieces — replication-level safety at ~1.5x storage | 💤 Low |
