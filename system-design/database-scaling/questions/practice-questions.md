# Database Scaling · Practice Questions

[← Back to Database Scaling](../README.md) · [Q&A with answers →](database-scaling-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](database-scaling-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**27 questions.**

---

## 01 · Indexing

- [ ] 🟢 Q1. What is a database index and what does it cost?
- [ ] 🟡 Q2. You have an index on (user_id, created_at). Which queries can use it?
- [ ] 🟡 Q3. What is a covering index?
- [ ] 🔴 Q4. You added an index but the query is still slow and EXPLAIN shows a full scan. Why?

---

## 02 · Partitioning

- [ ] 🟢 Q5. What's the difference between partitioning and sharding?
- [ ] 🟡 Q6. What is partition pruning and when does it fail?
- [ ] 🔴 Q7. Deleting old rows from a 2-billion-row logs table takes hours and locks things up. What do you do?

---

## 03 · Replication

- [ ] 🟢 Q8. Why replicate a database, and does it help with write scaling?
- [ ] 🟡 Q9. Compare leader-follower, multi-leader, and leaderless replication.
- [ ] 🟡 Q10. What does W + R > N mean in quorum replication?
- [ ] 🔴 Q11. A user updates their profile, refreshes, and sees the old data. What's happening and how do you fix it?

---

## 04 · Sharding

- [ ] 🟢 Q12. What is sharding and when should you do it?
- [ ] 🟡 Q13. How do you choose a shard key?
- [ ] 🟡 Q14. Compare range, hash, and directory-based sharding.
- [ ] 🔴 Q15. One shard is at 100% CPU while the others are idle. What do you do?

---

## 05 · Query Optimization

- [ ] 🟢 Q16. How do you find and fix a slow query?
- [ ] 🟡 Q17. What is the N+1 query problem?
- [ ] 🔴 Q18. Infinite scroll gets slower the further users scroll, and sometimes shows duplicates. Why?

---

## 06 · Denormalization

- [ ] 🟢 Q19. What is denormalization and why would you do it?
- [ ] 🟡 Q20. How do you keep denormalized copies consistent?
- [ ] 🔴 Q21. A viral post's like counter is causing lock contention and slow writes. Fix it.

---

## 07 · Materialized Views

- [ ] 🟢 Q22. What's the difference between a view and a materialized view?
- [ ] 🟡 Q23. How do you keep a materialized view fresh?
- [ ] 🔴 Q24. A dashboard query takes 10 seconds and runs on every page load. What do you do?

---

## 08 · Connection Pooling

- [ ] 🟢 Q25. Why use a connection pool?
- [ ] 🟡 Q26. How big should a connection pool be?
- [ ] 🔴 Q27. After moving to serverless, the database starts rejecting connections during traffic spikes. Why, and how do you fix it?
