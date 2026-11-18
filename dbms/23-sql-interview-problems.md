# 23 · Classic SQL Interview Problems

> [!info] Topic info
> **Subject:** DBMS · **Part:** Transactions & Internals · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to DBMS](README.md) · [Interview questions →](questions/transactions-and-internals-qa.md)

> [!abstract] In one line
> Most SQL interview questions are a handful of **patterns** — **window functions** (rank, running total, gaps-and-islands), **GROUP BY + HAVING**, **self joins**, and **CASE pivots** — learn the pattern, not the answer.

---

## 1. The setup

Sample tables used below (Postgres syntax; MySQL 8 differences noted):

- `employees(id, name, salary, dept_id, manager_id)`
- `users(id, email)` · `logins(user_id, login_date)` · `orders(id, customer_id, amount, created_at)` · `sales(product, month, amount)`

Background: [Joins](06-joins.md), [GROUP BY](08-group-by-and-aggregates.md), [Window Functions](09-window-functions.md), [CTEs](07-subqueries-and-ctes.md).

---

## 2. Ranking problems

### Nth highest salary (say N = 3)

```sql
SELECT DISTINCT salary FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 2;            -- OFFSET N-1
```
Window version (handles ties cleanly, easy to extend):
```sql
SELECT salary FROM (
  SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk FROM employees
) t WHERE rnk = 3 LIMIT 1;
```

> [!note] ROW_NUMBER vs RANK vs DENSE_RANK
> Salaries 100, 100, 90 → ROW_NUMBER 1,2,3 · RANK 1,1,3 · DENSE_RANK 1,1,2. "Nth highest salary" means **distinct** values → `DENSE_RANK`.

### Top N per group (top 3 earners per department)

```sql
SELECT dept_id, name, salary FROM (
  SELECT *, DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk
  FROM employees
) t
WHERE rnk <= 3;
```

### Second highest salary per department

Same pattern — just `WHERE rnk = 2`. Departments with only one salary return nothing (say this out loud).

---

## 3. Duplicates

### Find duplicates

```sql
SELECT email, COUNT(*) AS cnt
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

### Delete duplicates (keep the lowest id)

```sql
DELETE FROM users WHERE id IN (
  SELECT id FROM (
    SELECT id, ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) AS rn FROM users
  ) t WHERE rn > 1
);
```
Shorter self-join version — Postgres: `DELETE FROM users a USING users b WHERE a.email = b.email AND a.id > b.id;` · MySQL: `DELETE a FROM users a JOIN users b ON a.email = b.email AND a.id > b.id;`

> [!warning] MySQL gotcha
> MySQL won't let a subquery select from the table you're deleting from — wrapping it in a derived table (`SELECT id FROM (...) t`) like above gets around that.

---

## 4. Self join — earning more than their manager

```sql
SELECT e.name AS employee
FROM employees e
JOIN employees m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```
Same table twice, two aliases. Use `LEFT JOIN` if you also need people **without** a manager.

---

## 5. Running total

```sql
SELECT customer_id, created_at, amount,
       SUM(amount) OVER (PARTITION BY customer_id ORDER BY created_at
                         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM orders;
```
Without `ROWS …`, the default frame is `RANGE`, so rows with the **same timestamp** get added together. Swap `SUM` for `AVG` with `ROWS BETWEEN 6 PRECEDING AND CURRENT ROW` → 7-day moving average.

---

## 6. Consecutive days / streaks (gaps and islands)

Trick: for consecutive dates, `date − row_number` stays **constant**. Group by that value.

```sql
WITH d AS (SELECT DISTINCT user_id, login_date FROM logins),
g AS (
  SELECT user_id, login_date,
         login_date - CAST(ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) AS int) AS grp
  FROM d
)
SELECT user_id, MIN(login_date) AS streak_start, COUNT(*) AS days
FROM g GROUP BY user_id, grp
HAVING COUNT(*) >= 3;            -- users with 3+ day streaks
```

| login_date | row_number | date − rn |
|---|---|---|
| **Jan 1** | 1 | Dec 31 |
| **Jan 2** | 2 | Dec 31 ← same island |
| **Jan 5** | 3 | Jan 2 ← new island |

MySQL: use `DATE_SUB(login_date, INTERVAL rn DAY)`.

---

## 7. Pivot — rows into columns

```sql
SELECT product,
       SUM(CASE WHEN month = 'Jan' THEN amount ELSE 0 END) AS jan,
       SUM(CASE WHEN month = 'Feb' THEN amount ELSE 0 END) AS feb,
       SUM(CASE WHEN month = 'Mar' THEN amount ELSE 0 END) AS mar
FROM sales
GROUP BY product;
```
Conditional aggregation works in every database. Postgres also has `SUM(amount) FILTER (WHERE month = 'Jan')`.

> [!tip] 🎤 Say it like this
> "Before writing SQL I name the pattern. 'Top N per group' or 'Nth highest' is DENSE_RANK with PARTITION BY. 'Duplicates' is GROUP BY with HAVING, and deleting them is ROW_NUMBER greater than 1. 'Compare a row with another row in the same table' is a self join. 'Consecutive days' is gaps and islands — date minus row number."

> [!failure] Interview traps
> - Using `ROW_NUMBER` for "Nth highest salary" — ties give the wrong answer; use `DENSE_RANK`.
> - Filtering a window function in `WHERE` — not allowed; wrap it in a subquery/CTE first.
> - `NOT IN` with a subquery that returns a `NULL` → returns **zero rows**. Prefer `NOT EXISTS`.
> - Forgetting `DISTINCT` dates in streak problems — two logins on one day break the math.

---

## Summary

| Problem | Pattern |
|---|---|
| **Nth highest / top N per group** | `DENSE_RANK() OVER (PARTITION BY … ORDER BY …)` |
| **Find duplicates** | `GROUP BY … HAVING COUNT(*) > 1` |
| **Delete duplicates** | `ROW_NUMBER() … rn > 1` or self-join delete |
| **Earns more than manager** | Self join |
| **Running total** | `SUM() OVER (ORDER BY … ROWS …)` |
| **Streaks** | Gaps and islands: `date − ROW_NUMBER()` |
| **Pivot** | `SUM(CASE WHEN …)` + `GROUP BY` |

> [!success] Key takeaway
> Seven patterns cover most SQL rounds. Say the pattern name first, then write the query — interviewers reward the reasoning as much as the syntax.
