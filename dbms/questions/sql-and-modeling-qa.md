# Q&A · DBMS — SQL & Data Modeling

[← Back to DBMS](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 01 · What is a DBMS

[Read the notes →](../01-what-is-dbms.md)

> [!question]- 🟢 Q1. What is a DBMS and why not just use files?
> A DBMS is software that stores and manages data and lets many users query and change it safely. Plain files give you redundancy, inconsistent copies, lost updates when two writers collide, and half-written data after a crash. A DBMS adds a query language, constraints, concurrency control, crash recovery and permissions.

> [!question]- 🟢 Q2. What's the difference between DDL, DML, DCL and TCL?
> DDL defines structure — CREATE, ALTER, DROP. DML works with rows — SELECT, INSERT, UPDATE, DELETE. DCL handles permissions with GRANT and REVOKE, and TCL controls transactions with COMMIT, ROLLBACK and SAVEPOINT.

> [!question]- 🟡 Q3. What is data independence?
> It's the ability to change one level of the database without breaking the level above. Physical independence means I can add an index or move storage without changing queries. Logical independence means I can change the table structure, like adding a column, without breaking apps that use views.

---

## 02 · ER Model

[Read the notes →](../02-er-model.md)

> [!question]- 🟢 Q4. What are entities, attributes and relationships?
> An entity is a thing we store, like User or Post. Attributes are its properties, like name or created_at. A relationship connects entities, like "User writes Post", and has a cardinality such as one-to-many.

> [!question]- 🟡 Q5. How do you convert 1:N and M:N relationships into tables?
> For one-to-many, put a foreign key on the many side — `posts.user_id` points to users. For many-to-many, like users liking posts, create a junction table `likes(user_id, post_id)` with both foreign keys and a composite primary key on the pair.

> [!question]- 🟡 Q6. What is a weak entity?
> An entity that can't be uniquely identified on its own and depends on an owner entity. For example, "comment number 3" only makes sense inside a specific post, so its key is the post's key plus its own partial key. If the owner is deleted, the weak entity goes too.

---

## 03 · Relational Model & Keys

[Read the notes →](../03-relational-model-and-keys.md)

> [!question]- 🟢 Q7. Explain super key, candidate key, primary key and alternate key.
> A super key is any set of columns that uniquely identifies a row. A candidate key is a minimal super key — remove any column and it's no longer unique. The primary key is the candidate we choose, and the remaining candidates are alternate keys, enforced with UNIQUE.

> [!question]- 🟢 Q8. Primary key vs unique key?
> A primary key is unique and NOT NULL, and a table has only one. A unique key also prevents duplicates, but usually allows NULLs and a table can have many. Both create an index behind the scenes.

> [!question]- 🟡 Q9. Surrogate key or natural key — which do you prefer?
> Usually a surrogate key like a BIGINT or UUID as the primary key, because natural values like email can change and are longer to index and join on. I still keep the natural key unique with a UNIQUE constraint so duplicates can't sneak in.

> [!question]- 🔴 Q10. A user is deleted but their posts still show up with a broken author. How do you prevent this at the database level?
> Add a foreign key from `posts.user_id` to `users.user_id` so referential integrity is enforced. Then choose the delete behavior: ON DELETE CASCADE removes the posts, SET NULL keeps them as "deleted user", and RESTRICT blocks the delete. Many apps soft-delete users instead, so history stays intact.

---

## 04 · Relational Algebra

[Read the notes →](../04-relational-algebra.md)

> [!question]- 🟢 Q11. What's the difference between selection and projection?
> Selection (σ) filters rows, like a WHERE clause. Projection (π) picks columns, like the SELECT list, and in pure algebra it also removes duplicates. A common confusion is that SQL's SELECT keyword is actually projection.

> [!question]- 🟡 Q12. Why does relational algebra matter for query performance?
> The database converts SQL into a tree of algebra operators and the optimizer rearranges it into an equivalent but cheaper plan. The classic rule is pushing selections down so rows are filtered before joins, which shrinks the data being joined.

---

## 05 · SQL Basics

[Read the notes →](../05-sql-basics.md)

> [!question]- 🟢 Q13. What is the logical order of execution of a SQL query?
> FROM and JOINs first, then WHERE, GROUP BY, HAVING, then SELECT, DISTINCT, ORDER BY, and finally LIMIT. That's why I can't use a select alias or an aggregate in WHERE, but I can use the alias in ORDER BY.

> [!question]- 🟢 Q14. DELETE vs TRUNCATE vs DROP?
> DELETE removes chosen rows with a WHERE clause, row by row, and fires triggers. TRUNCATE quickly empties the whole table and resets identity counters. DROP removes the table itself, structure and all.

> [!question]- 🟡 Q15. Why does `WHERE col = NULL` return nothing?
> NULL means unknown, so comparing anything with NULL gives unknown, not true, and the row is filtered out. You need `IS NULL` or `IS NOT NULL`. This is also why `NOT IN` against a list containing a NULL returns zero rows.

> [!question]- 🟡 Q16. UNION vs UNION ALL?
> UNION combines results and removes duplicates, which costs an extra sort or hash step. UNION ALL keeps every row and is faster. I use UNION ALL unless I actually need deduplication.

---

## 06 · Joins

[Read the notes →](../06-joins.md)

> [!question]- 🟢 Q17. Explain inner, left, right and full outer joins.
> Inner join returns only rows that match on both sides. Left join keeps all rows from the left table and fills NULLs where the right has no match; right join is the mirror. Full outer join keeps all rows from both sides, with NULLs where there's no match.

> [!question]- 🟡 Q18. How do you find users who have never posted?
> Anti-join: LEFT JOIN users to posts on user_id and keep rows where `p.post_id IS NULL`. Alternatively `WHERE NOT EXISTS (SELECT 1 FROM posts p WHERE p.user_id = u.user_id)`. I avoid NOT IN in case posts.user_id has NULLs.
> ```sql
> SELECT u.name FROM users u
> LEFT JOIN posts p ON p.user_id = u.user_id
> WHERE p.post_id IS NULL;
> ```

> [!question]- 🟡 Q19. What is a self join? Give an example.
> A self join joins a table with itself using two aliases. In a follows table, to find mutual follows I join `follows a` with `follows b` where a's follower is b's followee and a's followee is b's follower. Employee-manager lookups are the other classic case.

> [!question]- 🔴 Q20. A report's like counts are 10x too high after someone added a join to comments. Why?
> Row explosion: each post row is multiplied by its comments, so every like is counted once per comment. Fix it by aggregating likes and comments separately in subqueries or CTEs and then joining the totals, or by using COUNT(DISTINCT like_id) if that's enough.

---

## 07 · Subqueries & CTEs

[Read the notes →](../07-subqueries-and-ctes.md)

> [!question]- 🟢 Q21. What is a correlated subquery?
> A subquery that references a column from the outer query, so logically it runs once per outer row. For example, EXISTS checking for a comment by the same user. It can be slow on big tables unless the optimizer turns it into a join, so I check the plan.

> [!question]- 🟡 Q22. CTE vs subquery — what's the difference, and is a CTE faster?
> A CTE is a named subquery declared with WITH, so I can build a query in readable steps and reuse a step more than once. It's mostly about readability, not speed — modern PostgreSQL inlines CTEs like subqueries. Older versions always materialized them, which could change performance either way.

> [!question]- 🔴 Q23. How would you fetch an entire nested comment thread in one query?
> Use a recursive CTE. The anchor selects the root comment, and the recursive part joins comments whose parent_id matches a row already in the result, adding depth + 1. It stops when no new rows appear; on data that can have cycles I add a depth limit.

---

## 08 · GROUP BY & Aggregates

[Read the notes →](../08-group-by-and-aggregates.md)

> [!question]- 🟢 Q24. WHERE vs HAVING?
> WHERE filters individual rows before grouping, so it can't use aggregates. HAVING filters groups after GROUP BY, so it can use COUNT or SUM. If a condition doesn't need an aggregate, it belongs in WHERE because it shrinks the data early.

> [!question]- 🟢 Q25. COUNT(*) vs COUNT(col) vs COUNT(DISTINCT col)?
> COUNT(*) counts all rows. COUNT(col) counts rows where col isn't NULL. COUNT(DISTINCT col) counts unique non-NULL values — useful after joins that duplicate rows.

> [!question]- 🟡 Q26. Why does `SELECT user_id, content, COUNT(*) FROM posts GROUP BY user_id` fail?
> After grouping there's one row per user, but each user has many posts, so the database can't know which content to show. Every selected column must be in GROUP BY or inside an aggregate. If I need detail rows plus a total, I use a window function instead.

> [!question]- 🔴 Q27. A "posts per user" report is missing users with zero posts. How do you fix it?
> The query probably starts from posts or uses an inner join, so users with no posts never appear. Start from users, LEFT JOIN posts, and use COUNT(p.post_id) rather than COUNT(*) — COUNT(*) would count the NULL row and show 1 instead of 0.

---

## 09 · Window Functions

[Read the notes →](../09-window-functions.md)

> [!question]- 🟢 Q28. What is a window function and how is it different from GROUP BY?
> A window function computes a value over a set of related rows but keeps every row, while GROUP BY collapses each group into one row. It's defined with OVER, using PARTITION BY to split groups and ORDER BY to order within them.

> [!question]- 🟡 Q29. ROW_NUMBER vs RANK vs DENSE_RANK?
> For values 90, 80, 80, 70: ROW_NUMBER gives 1, 2, 3, 4 — always unique. RANK gives 1, 2, 2, 4 — ties share and then a gap. DENSE_RANK gives 1, 2, 2, 3 — no gap. For "Nth highest value" I use DENSE_RANK.

> [!question]- 🟡 Q30. How do you get each user's top 3 most-liked posts?
> Rank posts per user with ROW_NUMBER partitioned by user_id and ordered by likes descending, inside a CTE, then filter rn <= 3 outside. You can't put a window function in WHERE because it's computed after WHERE.
> ```sql
> WITH r AS (SELECT *, ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY likes DESC) rn FROM post_stats)
> SELECT * FROM r WHERE rn <= 3;
> ```

> [!question]- 🔴 Q31. Product wants daily signups with day-over-day change and a running total. How would you write it?
> Use LAG(signups) OVER (ORDER BY day) to get yesterday's value and subtract it for the change. Use SUM(signups) OVER (ORDER BY day) for the running total. For a 7-day moving average I'd add a frame: ROWS BETWEEN 6 PRECEDING AND CURRENT ROW.

---

## 10 · Views, Procedures & Triggers

[Read the notes →](../10-views-procedures-triggers.md)

> [!question]- 🟢 Q32. What is a view and why use one?
> A view is a saved query that behaves like a table. It simplifies repeated joins, hides sensitive columns like email from some users, and gives apps a stable interface while the underlying tables change. A normal view stores no data, so it isn't faster by itself.

> [!question]- 🟡 Q33. View vs materialized view?
> A view re-runs its query every time, so it's always fresh but no faster. A materialized view stores the result, so reads are fast, but the data is stale until it's refreshed. I use materialized views for heavy dashboards that can tolerate a few minutes of delay.

> [!question]- 🔴 Q34. Bulk importing 1M likes takes hours, though single inserts are fast. What do you check?
> I'd check for row-level triggers on the likes table — a trigger updating a counter or writing an audit row runs 1M times. Also foreign key checks and indexes on each insert. Options are to disable or batch the trigger work during import, use COPY, and recompute counters once at the end.

---

## 11 · Functional Dependencies

[Read the notes →](../11-functional-dependencies.md)

> [!question]- 🟢 Q35. What is a functional dependency?
> A → B means knowing A fixes B: any two rows with the same A must have the same B. For example user_id → email. FDs come from business rules, not from the sample data you happen to see.

> [!question]- 🟡 Q36. How do you find a candidate key from a set of FDs?
> Compute the attribute closure: start with a set of attributes and keep adding whatever the FDs let you derive. If the closure covers every column, it's a super key, and if no smaller subset does, it's a candidate key. Attributes that never appear on the right side of any FD must be in every key.

---

## 12 · Normalization

[Read the notes →](../12-normalization.md)

> [!question]- 🟢 Q37. What is normalization and what anomalies does it fix?
> Normalization splits tables so each fact is stored once. It fixes update anomalies, where changing a name means editing many rows; insert anomalies, where you can't add a user without a post; and delete anomalies, where deleting the last post also loses the user.

> [!question]- 🟡 Q38. Explain 1NF, 2NF, 3NF and BCNF.
> 1NF means atomic values, no lists in a cell. 2NF removes partial dependencies, where a column depends on only part of a composite key. 3NF removes transitive dependencies, where a non-key column depends on another non-key column. BCNF is stricter: the left side of every dependency must be a super key.

> [!question]- 🟡 Q39. What's the difference between 3NF and BCNF?
> 3NF allows a dependency X → A when A is part of some candidate key, even if X isn't a super key. BCNF doesn't allow that. Going to BCNF can lose a dependency that you could otherwise enforce in one table, so many designs stop at 3NF.

> [!question]- 🔴 Q40. Your fully normalized feed query joins 4 tables and is too slow at 20k reads/s. What do you do?
> First check indexes and the query plan. If joins are still the bottleneck, denormalize the hot path: store like_count on posts and copy author name into a feed table or cache, updated on write or by a background job. Normalized tables stay the source of truth, and I accept slightly stale counts as the trade-off.
