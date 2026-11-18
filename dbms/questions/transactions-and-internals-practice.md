# DBMS — Transactions & Internals · Practice Questions

[← Back to DBMS](../README.md) · [Q&A with answers →](transactions-and-internals-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](transactions-and-internals-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**39 questions.**

---

## 13 · Transactions & ACID

- [ ] 🟢 Q1. What is a transaction and what does ACID stand for?
- [ ] 🟢 Q2. Which DB mechanism gives each ACID property?
- [ ] 🟡 Q3. Walk me through the states of a transaction.
- [ ] 🔴 Q4. Our checkout calls the payment gateway inside a DB transaction and the DB shows lots of lock waits under load. Why, and what would you change?

---

## 14 · Schedules & Serializability

- [ ] 🟢 Q5. What's the difference between a serial and a serializable schedule?
- [ ] 🟡 Q6. How do you check if a schedule is conflict serializable?
- [ ] 🔴 Q7. Conflict vs view serializability — which one do databases use and why?

---

## 15 · Locking & 2PL

- [ ] 🟢 Q8. What are shared and exclusive locks?
- [ ] 🟡 Q9. What is two-phase locking and why does it work?
- [ ] 🟡 Q10. Why do databases use strict 2PL instead of basic 2PL?
- [ ] 🔴 Q11. Two users booked the last seat on a flight. How do you fix it?

---

## 16 · Isolation Levels & MVCC

- [ ] 🟢 Q12. Name the four isolation levels and what each prevents.
- [ ] 🟡 Q13. How does MVCC work in Postgres?
- [ ] 🟡 Q14. What's write skew and which isolation level allows it?
- [ ] 🔴 Q15. A Postgres table keeps growing on disk even though row count is stable, and queries are slowing down. What's going on?

---

## 17 · Deadlocks

- [ ] 🟢 Q16. What is a deadlock and what conditions cause it?
- [ ] 🟡 Q17. How do databases detect deadlocks?
- [ ] 🟡 Q18. Explain wait-die vs wound-wait.
- [ ] 🔴 Q19. Your money-transfer API throws occasional deadlock errors under load. How do you fix it?

---

## 18 · Recovery & WAL

- [ ] 🟢 Q20. What is write-ahead logging?
- [ ] 🟡 Q21. What do undo and redo do during crash recovery?
- [ ] 🔴 Q22. Why do we need checkpoints, and what's the trade-off in how often they run?

---

## 19 · Indexing & B+ Trees

- [ ] 🟢 Q23. Clustered vs non-clustered index?
- [ ] 🟡 Q24. Why do databases use B+ trees instead of B trees?
- [ ] 🟡 Q25. When would you use a hash index over a B+ tree?
- [ ] 🔴 Q26. Inserts into an InnoDB table with a UUID primary key got slow as the table grew. Why?

---

## 20 · Storage & File Organization

- [ ] 🟢 Q27. What is a page and a buffer pool?
- [ ] 🟡 Q28. Heap file vs clustered organization — Postgres vs MySQL?

---

## 21 · Query Processing

- [ ] 🟢 Q29. What happens inside the DB when you run a query?
- [ ] 🟡 Q30. Compare nested loop, hash join and merge join.
- [ ] 🟡 Q31. Why might the optimizer choose a bad plan?
- [ ] 🔴 Q32. A report query was fast last month but now takes minutes after a big data import. What do you check?

---

## 22 · NoSQL Basics

- [ ] 🟢 Q33. What are the main types of NoSQL databases?
- [ ] 🟡 Q34. What is BASE and how is it different from ACID?
- [ ] 🔴 Q35. You're storing chat messages for 50M users. Postgres or Cassandra, and how would you model it?

---

## 23 · SQL Interview Problems

- [ ] 🟢 Q36. Find emails that appear more than once in the users table.
- [ ] 🟡 Q37. Find the second highest salary in each department.
- [ ] 🟡 Q38. Find employees who earn more than their manager.
- [ ] 🔴 Q39. Find users who logged in on 3 or more consecutive days.
