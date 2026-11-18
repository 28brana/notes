# DBMS — SQL & Data Modeling · Practice Questions

[← Back to DBMS](../README.md) · [Q&A with answers →](sql-and-modeling-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](sql-and-modeling-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**40 questions.**

---

## 01 · What is a DBMS

- [ ] 🟢 Q1. What is a DBMS and why not just use files?
- [ ] 🟢 Q2. What's the difference between DDL, DML, DCL and TCL?
- [ ] 🟡 Q3. What is data independence?

---

## 02 · ER Model

- [ ] 🟢 Q4. What are entities, attributes and relationships?
- [ ] 🟡 Q5. How do you convert 1:N and M:N relationships into tables?
- [ ] 🟡 Q6. What is a weak entity?

---

## 03 · Relational Model & Keys

- [ ] 🟢 Q7. Explain super key, candidate key, primary key and alternate key.
- [ ] 🟢 Q8. Primary key vs unique key?
- [ ] 🟡 Q9. Surrogate key or natural key — which do you prefer?
- [ ] 🔴 Q10. A user is deleted but their posts still show up with a broken author. How do you prevent this at the database level?

---

## 04 · Relational Algebra

- [ ] 🟢 Q11. What's the difference between selection and projection?
- [ ] 🟡 Q12. Why does relational algebra matter for query performance?

---

## 05 · SQL Basics

- [ ] 🟢 Q13. What is the logical order of execution of a SQL query?
- [ ] 🟢 Q14. DELETE vs TRUNCATE vs DROP?
- [ ] 🟡 Q15. Why does `WHERE col = NULL` return nothing?
- [ ] 🟡 Q16. UNION vs UNION ALL?

---

## 06 · Joins

- [ ] 🟢 Q17. Explain inner, left, right and full outer joins.
- [ ] 🟡 Q18. How do you find users who have never posted?
- [ ] 🟡 Q19. What is a self join? Give an example.
- [ ] 🔴 Q20. A report's like counts are 10x too high after someone added a join to comments. Why?

---

## 07 · Subqueries & CTEs

- [ ] 🟢 Q21. What is a correlated subquery?
- [ ] 🟡 Q22. CTE vs subquery — what's the difference, and is a CTE faster?
- [ ] 🔴 Q23. How would you fetch an entire nested comment thread in one query?

---

## 08 · GROUP BY & Aggregates

- [ ] 🟢 Q24. WHERE vs HAVING?
- [ ] 🟢 Q25. COUNT(*) vs COUNT(col) vs COUNT(DISTINCT col)?
- [ ] 🟡 Q26. Why does `SELECT user_id, content, COUNT(*) FROM posts GROUP BY user_id` fail?
- [ ] 🔴 Q27. A "posts per user" report is missing users with zero posts. How do you fix it?

---

## 09 · Window Functions

- [ ] 🟢 Q28. What is a window function and how is it different from GROUP BY?
- [ ] 🟡 Q29. ROW_NUMBER vs RANK vs DENSE_RANK?
- [ ] 🟡 Q30. How do you get each user's top 3 most-liked posts?
- [ ] 🔴 Q31. Product wants daily signups with day-over-day change and a running total. How would you write it?

---

## 10 · Views, Procedures & Triggers

- [ ] 🟢 Q32. What is a view and why use one?
- [ ] 🟡 Q33. View vs materialized view?
- [ ] 🔴 Q34. Bulk importing 1M likes takes hours, though single inserts are fast. What do you check?

---

## 11 · Functional Dependencies

- [ ] 🟢 Q35. What is a functional dependency?
- [ ] 🟡 Q36. How do you find a candidate key from a set of FDs?

---

## 12 · Normalization

- [ ] 🟢 Q37. What is normalization and what anomalies does it fix?
- [ ] 🟡 Q38. Explain 1NF, 2NF, 3NF and BCNF.
- [ ] 🟡 Q39. What's the difference between 3NF and BCNF?
- [ ] 🔴 Q40. Your fully normalized feed query joins 4 tables and is too slow at 20k reads/s. What do you do?
