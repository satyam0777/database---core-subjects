# SQL Interview Cheat Sheet 🎯

Quick reference for interviews - bookmark this!

---

## SYNTAX QUICK REFERENCE

### SELECT Basics
```sql
SELECT column1, column2 FROM table;
SELECT * FROM table;
SELECT DISTINCT column FROM table;
SELECT column AS alias FROM table;
```

### WHERE Conditions
```sql
WHERE column = value
WHERE column > 5
WHERE column BETWEEN 1 AND 10
WHERE column IN ('a', 'b', 'c')
WHERE column LIKE 'prefix%'
WHERE column IS NULL
WHERE column IS NOT NULL
WHERE condition1 AND condition2
WHERE condition1 OR condition2
```

### ORDER & LIMIT
```sql
ORDER BY column ASC / DESC
ORDER BY col1, col2  -- Multiple columns
LIMIT 10             -- First 10
LIMIT 10 OFFSET 5    -- Skip 5, take 10
```

---

## JOINs (Very Important!)

```sql
-- INNER JOIN: Only matching rows
SELECT * FROM a JOIN b ON a.id = b.a_id;

-- LEFT JOIN: All from left, matching from right
SELECT * FROM a LEFT JOIN b ON a.id = b.a_id;

-- RIGHT JOIN: All from right, matching from left
SELECT * FROM a RIGHT JOIN b ON a.id = b.a_id;

-- FULL OUTER JOIN: All from both
SELECT * FROM a FULL OUTER JOIN b ON a.id = b.a_id;

-- CROSS JOIN: Cartesian product (a × b)
SELECT * FROM a CROSS JOIN b;

-- SELF JOIN: Table joins itself
SELECT e1.name, e2.name FROM employees e1 
JOIN employees e2 ON e1.manager_id = e2.id;
```

**Visual:**
```
LEFT:          RIGHT:         INNER:         FULL:
a  b           a  b           a  b           a  b
1  1           1  1           1  1           1  1
2  2  2        2  2  2        2  2           2  2
3           3  3           3           3     3
4           4                4           4     4
```

---

## AGGREGATES & GROUP BY

### Aggregate Functions
```sql
COUNT(*)           -- Count rows
COUNT(column)      -- Count non-NULL
SUM(column)        -- Sum values
AVG(column)        -- Average
MIN(column)        -- Minimum
MAX(column)        -- Maximum
STDDEV(column)     -- Standard deviation
```

### GROUP BY
```sql
SELECT department, COUNT(*) as count
FROM employees
GROUP BY department;

-- Multiple columns
SELECT dept, year, COUNT(*) 
FROM employees
GROUP BY dept, year;

-- With HAVING (filter groups)
SELECT dept, AVG(salary) as avg_sal
FROM employees
GROUP BY dept
HAVING AVG(salary) > 50000;
```

**WHERE vs HAVING:**
- WHERE: Filters rows BEFORE grouping
- HAVING: Filters groups AFTER grouping

---

## SUBQUERIES

```sql
-- In WHERE
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- In SELECT
SELECT name, (SELECT COUNT(*) FROM posts WHERE user_id = u.id) as posts
FROM users u;

-- In FROM (derived table)
SELECT * FROM (
    SELECT id, name, salary FROM employees
    WHERE salary > 50000
) AS high_earners;

-- EXISTS
SELECT * FROM employees e
WHERE EXISTS (SELECT 1 FROM posts WHERE user_id = e.id);
```

---

## WINDOW FUNCTIONS

```sql
-- ROW_NUMBER: Unique sequential number
ROW_NUMBER() OVER (ORDER BY salary DESC)

-- RANK: Same rank for ties, skips numbers
RANK() OVER (ORDER BY salary DESC)

-- DENSE_RANK: Same rank for ties, no skips
DENSE_RANK() OVER (ORDER BY salary DESC)

-- PARTITION BY: Apply within groups
ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC)

-- LAG/LEAD: Access previous/next row
LAG(salary) OVER (ORDER BY salary)
LEAD(salary) OVER (ORDER BY salary)

-- Running total
SUM(salary) OVER (ORDER BY salary)

-- Ranking example:
SELECT 
    name, salary, dept,
    RANK() OVER (PARTITION BY dept ORDER BY salary DESC) as rank
FROM employees;
```

---

## CTEs (WITH Clause)

```sql
-- Simple CTE
WITH high_earners AS (
    SELECT name, salary FROM employees WHERE salary > 60000
)
SELECT * FROM high_earners ORDER BY salary DESC;

-- Multiple CTEs
WITH dept_avg AS (
    SELECT dept, AVG(salary) as avg_sal FROM employees GROUP BY dept
),
high_earners AS (
    SELECT name, salary, dept FROM employees WHERE salary > 70000
)
SELECT h.*, d.avg_sal FROM high_earners h
JOIN dept_avg d ON h.dept = d.dept;

-- Recursive CTE (hierarchy)
WITH RECURSIVE hierarchy AS (
    SELECT id, name, manager_id, 0 as level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    SELECT e.id, e.name, e.manager_id, h.level + 1
    FROM employees e
    JOIN hierarchy h ON e.manager_id = h.id
)
SELECT * FROM hierarchy;
```

---

## SET OPERATIONS

```sql
-- UNION: Combine, remove duplicates
SELECT name FROM employees
UNION
SELECT name FROM contractors;

-- UNION ALL: Combine, keep duplicates
SELECT name FROM employees
UNION ALL
SELECT name FROM contractors;

-- INTERSECT: Common rows
SELECT name FROM employees
INTERSECT
SELECT name FROM contractors;

-- EXCEPT: In first, not in second
SELECT name FROM employees
EXCEPT
SELECT name FROM contractors;
```

---

## DATA TYPES

```sql
INT / BIGINT / SMALLINT / TINYINT
FLOAT / DECIMAL(10,2)
CHAR(10) / VARCHAR(100) / TEXT
DATE / TIME / DATETIME / TIMESTAMP
BOOLEAN
```

---

## CONSTRAINTS

```sql
PRIMARY KEY         -- Unique identifier
FOREIGN KEY         -- Reference another table
UNIQUE              -- All values unique
NOT NULL            -- Cannot be null
DEFAULT             -- Default value
CHECK               -- Condition on column
```

---

## INSERT, UPDATE, DELETE

```sql
-- INSERT
INSERT INTO table (col1, col2) VALUES (val1, val2);
INSERT INTO table VALUES (val1, val2, val3);

-- Multiple rows
INSERT INTO table VALUES (val1, val2), (val3, val4);

-- INSERT from SELECT
INSERT INTO backup SELECT * FROM table WHERE condition;

-- UPDATE
UPDATE table SET col1 = val1 WHERE condition;
UPDATE table SET col1 = val1, col2 = val2 WHERE condition;
UPDATE table SET salary = salary * 1.1 WHERE dept = 'IT';

-- DELETE
DELETE FROM table WHERE condition;
DELETE FROM table;  -- Delete all!
```

---

## USEFUL FUNCTIONS

```sql
-- String functions
UPPER(column)       -- 'hello' → 'HELLO'
LOWER(column)       -- 'Hello' → 'hello'
LENGTH(column)      -- Length of string
SUBSTR(col, 1, 3)   -- Substring
CONCAT(col1, col2)  -- Combine strings
TRIM(column)        -- Remove spaces
REPLACE(col, 'a', 'b')  -- Replace text

-- Date functions
NOW()               -- Current date/time
DATE(column)        -- Extract date
YEAR(column)        -- Extract year
MONTH(column)       -- Extract month
DAY(column)         -- Extract day
DATE_ADD(date, INTERVAL 1 DAY)  -- Add 1 day
DATE_SUB(date, INTERVAL 1 MONTH) -- Subtract 1 month
DATEDIFF(date1, date2)  -- Days between

-- Math functions
ABS(number)         -- Absolute value
ROUND(number, 2)    -- Round to 2 decimals
CEIL(number)        -- Round up
FLOOR(number)       -- Round down
MOD(a, b)           -- Remainder (a % b)

-- CASE: Conditional logic
CASE 
    WHEN condition THEN value
    WHEN condition THEN value
    ELSE value
END
```

---

## COMMON PATTERNS

### Pattern 1: Top N Per Group
```sql
WITH ranked AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC) as rn
    FROM employees
)
SELECT * FROM ranked WHERE rn <= 3;
```

### Pattern 2: Running Total
```sql
SELECT 
    date,
    amount,
    SUM(amount) OVER (ORDER BY date) as running_total
FROM transactions;
```

### Pattern 3: Rank with Tie Breaking
```sql
SELECT 
    name,
    RANK() OVER (ORDER BY salary DESC, hire_date ASC) as rank
FROM employees;
```

### Pattern 4: Duplicate Rows
```sql
SELECT column, COUNT(*) FROM table
GROUP BY column
HAVING COUNT(*) > 1;
```

### Pattern 5: Before/After Comparison
```sql
SELECT 
    name,
    salary,
    LAG(salary) OVER (ORDER BY name) as prev_salary,
    salary - LAG(salary) OVER (ORDER BY name) as difference
FROM employees;
```

### Pattern 6: Consecutive Days/Months
```sql
WITH daily AS (
    SELECT date, ROW_NUMBER() OVER (ORDER BY date) as rn,
           DATE_SUB(date, INTERVAL ROW_NUMBER() OVER (ORDER BY date) DAY) as grp
    FROM events
)
SELECT grp, COUNT(*) as streak_length
FROM daily
GROUP BY grp
HAVING COUNT(*) >= 3;
```

---

## OPTIMIZATION TIPS

```sql
-- 1. Use specific columns, not *
SELECT id, name FROM users;  -- Good
SELECT * FROM users;          -- Avoid when possible

-- 2. Index on WHERE and JOIN columns
CREATE INDEX idx_name ON table(column);

-- 3. Use INNER JOIN instead of subquery
SELECT * FROM a WHERE id IN (SELECT id FROM b)  -- Slower
SELECT * FROM a JOIN b ON a.id = b.id           -- Faster

-- 4. Move conditions to WHERE, not HAVING
SELECT dept, AVG(salary) FROM emp WHERE salary > 50000 GROUP BY dept
-- Use WHERE for row-level conditions

-- 5. Use LIMIT to reduce rows
SELECT * FROM big_table LIMIT 100;

-- 6. Batch operations
INSERT INTO table VALUES (1,1), (2,2), (3,3);  -- One query

-- 7. Use EXPLAIN to debug
EXPLAIN SELECT * FROM table WHERE id = 1;
```

---

## COMMON MISTAKES

❌ Wrong:
```sql
SELECT name, salary, department FROM employees GROUP BY department;
-- Non-grouped column name - unpredictable result
```

✅ Correct:
```sql
SELECT department, AVG(salary) FROM employees GROUP BY department;
```

❌ Wrong:
```sql
SELECT * FROM employees WHERE AVG(salary) > 50000;
-- Can't use aggregate in WHERE
```

✅ Correct:
```sql
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

❌ Wrong:
```sql
DELETE FROM employees;  -- No WHERE - deletes everything!
```

✅ Correct:
```sql
-- Check first
SELECT * FROM employees WHERE salary < 30000;
-- Then delete
DELETE FROM employees WHERE salary < 30000;
```

---

## INTERVIEW POWER MOVES

### Power Move 1: Mention Indexes
```
"I'd add an index on department_id for faster JOINs"
```

### Power Move 2: Discuss Trade-offs
```
"This approach is O(n log n) but more readable than the subquery"
```

### Power Move 3: Ask for Edge Cases
```
"Should I handle NULL salaries? How?"
```

### Power Move 4: Trace Through Example
```
"Let me trace through with sample data: [walk through]"
```

### Power Move 5: Consider Performance
```
"For 1M rows, I'd use window functions instead of GROUP BY"
```

---

## 30-SECOND ANSWERS

**Q: Difference between WHERE and HAVING?**
```
WHERE filters rows BEFORE grouping.
HAVING filters groups AFTER grouping.
```

**Q: INNER vs LEFT JOIN?**
```
INNER: Only rows in both tables.
LEFT: All rows from left + matches from right.
```

**Q: What's a window function?**
```
Applies aggregates without collapsing rows.
Like comparing each row to others in its group.
```

**Q: How do you find duplicates?**
```sql
SELECT column, COUNT(*) FROM table
GROUP BY column
HAVING COUNT(*) > 1;
```

**Q: Optimize for 1M rows?**
```
Add indexes on WHERE and JOIN columns.
Use window functions instead of subqueries.
```

---

## RESOURCES TO PRACTICE

1. **LeetCode SQL**: 50 problems, medium difficulty
2. **HackerRank SQL**: Good tutorials + problems
3. **SQL Zoo**: Interactive learning
4. **Mode Analytics**: SQL tutorial + practice
5. **Your Own Database**: Create tables and experiment

---

## STUDY SCHEDULE (2 Weeks)

**Week 1:**
- Day 1-2: Fundamentals (CRUD, WHERE, ORDER BY)
- Day 3-4: JOINs (all types)
- Day 5: GROUP BY, HAVING
- Day 6: Subqueries
- Day 7: Review + 5 easy problems

**Week 2:**
- Day 1-2: Window Functions
- Day 3: CTEs
- Day 4: Set Operations
- Day 5: Practice 10 medium problems
- Day 6: Practice 5 hard problems
- Day 7: Mock interview

---

## CONFIDENCE CHECKLIST

✅ I can write CRUD operations
✅ I understand all JOIN types
✅ I can use GROUP BY with aggregates
✅ I can write subqueries
✅ I understand window functions
✅ I can optimize queries
✅ I can design a simple database
✅ I can explain my solution clearly

If ALL checked → Ready for interview! 🚀

---

## Last Minute Tips

- **Before Interview**: Review this sheet 1 hour before
- **During Interview**: Think out loud, ask clarifying questions
- **If Stuck**: Trace through example, simplify, build up
- **Mistakes**: Fix calmly, explain your thinking
- **Finish Early**: Discuss optimization, edge cases

---

**You've got this! 💪 Good luck! 🚀**
