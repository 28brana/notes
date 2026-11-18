# Databases · Practice Questions

[← Back to Databases](../README.md) · [Q&A with answers →](databases-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](databases-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**40 questions.**

---

## 01 · Database Types

- [ ] 🟢 Q1. Name the main types of databases and one use case for each.
- [ ] 🟡 Q2. How do you decide which database to use?
- [ ] 🔴 Q3. Your design uses Postgres, Redis, Elasticsearch, and Cassandra. The interviewer asks "isn't that too many?" How do you respond?

---

## 02 · SQL vs NoSQL

- [ ] 🟢 Q4. What are the main differences between SQL and NoSQL databases?
- [ ] 🟡 Q5. What does "model your queries first" mean in NoSQL?
- [ ] 🟡 Q6. Is it true that SQL databases can't scale?
- [ ] 🔴 Q7. You're designing a chat app with billions of messages a day. Which database for messages, and which for user accounts?

---

## 03 · ACID Transactions

- [ ] 🟢 Q8. What does ACID stand for?
- [ ] 🟡 Q9. Explain dirty read, non-repeatable read, and phantom read.
- [ ] 🟡 Q10. What isolation level do most databases use by default, and what does it allow?
- [ ] 🔴 Q11. Two users like a post at the same time and the count goes up by only 1. Why, and how do you fix it?

---

## 04 · B-Trees

- [ ] 🟢 Q12. What is a B-tree and why do databases use it?
- [ ] 🟡 Q13. Why is a B-tree slower for writes than an LSM tree?
- [ ] 🔴 Q14. Inserts into a big table got slower over time after you switched to random UUID primary keys. Why?

---

## 05 · LSM Trees

- [ ] 🟢 Q15. How does an LSM tree handle a write?
- [ ] 🟡 Q16. How do LSM trees keep reads fast if data is spread across many files?
- [ ] 🔴 Q17. A Cassandra cluster's read latency keeps rising after a big bulk delete. What's going on?

---

## 06 · Relational Databases

- [ ] 🟢 Q18. What are primary keys and foreign keys?
- [ ] 🟡 Q19. What is normalization, and when would you denormalize?
- [ ] 🟡 Q20. What is the N+1 query problem?
- [ ] 🔴 Q21. Your Postgres primary is at 90% CPU. What do you try, in order, before sharding?

---

## 07 · Document Databases

- [ ] 🟢 Q22. What is a document database and when would you use it?
- [ ] 🟡 Q23. When do you embed data in a document versus reference it?

---

## 08 · Key-Value Stores

- [ ] 🟢 Q24. What is a key-value store and what is it good for?
- [ ] 🟡 Q25. Redis vs DynamoDB — when do you use which?
- [ ] 🟡 Q26. Why does the choice of partition key matter so much in DynamoDB?
- [ ] 🔴 Q27. A celebrity posts and one Redis node hits 100% CPU while others are idle. What happened and how do you fix it?

---

## 09 · Full-Text Search

- [ ] 🟢 Q28. What is an inverted index?
- [ ] 🟡 Q29. Why not just use SQL LIKE '%word%' for search?
- [ ] 🔴 Q30. A user creates a post but can't find it in search for a few seconds. Is this a bug?

---

## 10 · Wide-Column Databases

- [ ] 🟢 Q31. What are the partition key and clustering key in Cassandra?
- [ ] 🟡 Q32. How does Cassandra give you consistent reads if it's leaderless?
- [ ] 🔴 Q33. Writes in your Cassandra cluster all hit one node. Your partition key is the date. What's wrong?

---

## 11 · Time-Series Databases

- [ ] 🟢 Q34. What makes a time-series database different from a normal database?
- [ ] 🔴 Q35. Your metrics DB's memory usage exploded after someone added a user_id tag. Why?

---

## 12 · Graph Databases

- [ ] 🟢 Q36. When would you choose a graph database over SQL?
- [ ] 🟡 Q37. Should a social network store its follow graph in a graph database?

---

## 13 · Vector Databases

- [ ] 🟢 Q38. What is an embedding and what does a vector database do with it?
- [ ] 🟡 Q39. What is approximate nearest neighbor search, and how does HNSW work at a high level?
- [ ] 🔴 Q40. You upgraded to a better embedding model and search quality dropped badly. Why?
