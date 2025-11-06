# Real Interview Problems & Solutions 💼

## Table of Contents
1. [Easy Problems](#easy-problems)
2. [Medium Problems](#medium-problems)
3. [Hard Problems](#hard-problems)
4. [How to Answer Interview Questions](#how-to-answer-interview-questions)

---

## Easy Problems

### Problem 1: Second Highest Salary

**Problem Statement:**
Find the second highest salary in the employee table. If there's no second highest, return NULL.

**Sample Data:**
```
employees:
id | name   | salary
1  | Raj    | 70000
2  | Priya  | 65000
3  | Amit   | 70000
4  | Sara   | 60000
```

**Expected Output:** 65000

**Approach:**
1. Remove duplicates (DISTINCT)
2. Sort descending
3. Skip first (OFFSET 1)
4. Take one (LIMIT 1)

**Solution 1: Using LIMIT & OFFSET**
```sql
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```

**Solution 2: Using MAX and Subquery**
```sql
SELECT MAX(salary)
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

**Solution 3: Using DENSE_RANK() (Best for Interviews)**
```sql
WITH ranked AS (
    SELECT salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) as rank
    FROM employees
)
SELECT salary
FROM ranked
WHERE rank = 2;
```

**Why Solution 3 is best:**
- Handles duplicate salaries correctly
- Scalable (can easily ask for 3rd, 4th highest)
- Shows you understand window functions
- Easy to explain

**Interview Answer Template:**
```
"I would use DENSE_RANK() to handle duplicates correctly.
- DENSE_RANK() assigns same rank to same salaries
- ORDER BY salary DESC to get highest first
- Filter WHERE rank = 2
- This is better than LIMIT/OFFSET because it handles ties"
```

---

### Problem 2: Employees with Duplicate Names

**Problem:** Find all employees who share the same name.

**Sample Data:**
```
employees:
id | name   | department
1  | John   | IT
2  | John   | HR
3  | Priya  | IT
```

**Expected Output:**
```
name   | count
John   | 2
```

**Solution:**
```sql
SELECT name, COUNT(*) as count
FROM employees
GROUP BY name
HAVING COUNT(*) > 1;
```

**Explanation:**
- GROUP BY name: Group all rows by name
- COUNT(*) > 1: Only groups with more than 1 person
- This filters duplicate names

---

### Problem 3: Department with Most Employees

**Problem:** Find the department with the most employees.

**Solution:**
```sql
SELECT department, COUNT(*) as emp_count
FROM employees
GROUP BY department
ORDER BY emp_count DESC
LIMIT 1;
```

**Better Solution (Handles Ties):**
```sql
WITH dept_count AS (
    SELECT department, COUNT(*) as emp_count
    FROM employees
    GROUP BY department
)
SELECT department, emp_count
FROM dept_count
WHERE emp_count = (SELECT MAX(emp_count) FROM dept_count);
```

---

## Medium Problems

### Problem 4: Join & Aggregate - Department Average Salary

**Problem:** For each department, show:
- Department name
- Average salary
- Number of employees
- Highest salary

**Sample Data:**
```
employees:
id | name   | salary | dept_id
1  | Raj    | 70000  | 1
2  | Priya  | 65000  | 1
3  | Amit   | 60000  | 2

departments:
dept_id | dept_name
1       | IT
2       | HR
```

**Solution:**
```sql
SELECT 
    d.dept_name,
    COUNT(e.id) as emp_count,
    AVG(e.salary) as avg_salary,
    MAX(e.salary) as max_salary,
    MIN(e.salary) as min_salary
FROM departments d
LEFT JOIN employees e ON d.dept_id = e.dept_id
GROUP BY d.dept_id, d.dept_name
ORDER BY avg_salary DESC;
```

**Why LEFT JOIN?**
- Includes departments with no employees (shows complete picture)
- INNER JOIN would miss empty departments

**Interview Explanation:**
```
"I use LEFT JOIN to ensure all departments are included,
even if they have no employees. Then I GROUP BY department
and apply aggregates to get the statistics you want."
```

---

### Problem 5: Employees Earning More Than Manager

**Problem:** Find employees who earn more than their manager.

**Sample Data:**
```
employees:
id | name   | salary | manager_id
1  | CEO    | 100000 | NULL
2  | Raj    | 70000  | 1
3  | Priya  | 75000  | 1
```

**Expected:** Priya earns 75000 > CEO's 100000? No. So no result if data is correct.

Let's say Priya's manager is Raj (manager_id = 2):
```
id | name   | salary | manager_id
3  | Priya  | 75000  | 2
```

Then Priya earns 75000 > Raj's 70000.

**Solution: Self Join**
```sql
SELECT 
    e.name as employee_name,
    e.salary as employee_salary,
    m.name as manager_name,
    m.salary as manager_salary
FROM employees e
JOIN employees m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

**Interview Tip:**
- Explain self join: "I join employees table with itself"
- "e is the employee record, m is the manager record"
- "WHERE filters for salary comparison"

---

### Problem 6: Consecutive Days Worked

**Problem:** For each employee, find the longest streak of consecutive days worked.

**Sample Data:**
```
attendance:
employee_id | work_date
1           | 2023-01-01
1           | 2023-01-02
1           | 2023-01-03
1           | 2023-01-05  -- Gap, not consecutive
1           | 2023-01-06
```

**Solution:**
```sql
WITH daily_work AS (
    SELECT 
        employee_id,
        work_date,
        ROW_NUMBER() OVER (PARTITION BY employee_id ORDER BY work_date) as rn,
        DATE_SUB(work_date, INTERVAL ROW_NUMBER() OVER (PARTITION BY employee_id ORDER BY work_date) DAY) as grp
    FROM attendance
),
streaks AS (
    SELECT 
        employee_id,
        grp,
        COUNT(*) as streak_length
    FROM daily_work
    GROUP BY employee_id, grp
)
SELECT 
    employee_id,
    MAX(streak_length) as longest_streak
FROM streaks
GROUP BY employee_id;
```

**How it works:**
1. ROW_NUMBER() assigns sequential numbers
2. Subtract this from dates to create groups
3. Consecutive dates have same group
4. COUNT per group gives streak length
5. MAX gives longest

**For Interview:**
```
"This is a tricky problem. The key insight is subtracting
the ROW_NUMBER from the date - consecutive dates will have
the same difference, creating groups. Then I count per group."
```

---

### Problem 7: Top N Employees Per Department

**Problem:** Get top 3 highest-paid employees from each department.

**Solution:**
```sql
WITH ranked_emp AS (
    SELECT 
        name,
        salary,
        department,
        RANK() OVER (PARTITION BY department ORDER BY salary DESC) as rank
    FROM employees
)
SELECT name, salary, department
FROM ranked_emp
WHERE rank <= 3
ORDER BY department, rank;
```

**Using CTE makes it:**
- Easy to read
- Easy to modify (change 3 to 5)
- Impresses interviewer

---

## Hard Problems

### Problem 8: Find Active Users

**Problem:** Find users who were active in at least 3 out of the last 7 days.

**Sample Data:**
```
activity:
user_id | activity_date
1       | 2023-01-01
1       | 2023-01-02
1       | 2023-01-04
1       | 2023-01-06
2       | 2023-01-02
2       | 2023-01-03
```

**Solution:**
```sql
SELECT DISTINCT user_id
FROM activity
WHERE activity_date > DATE_SUB(CURDATE(), INTERVAL 7 DAY)
GROUP BY user_id
HAVING COUNT(DISTINCT activity_date) >= 3;
```

**Breakdown:**
1. Filter last 7 days: `activity_date > DATE_SUB(CURDATE(), INTERVAL 7 DAY)`
2. Group by user_id
3. Count distinct days
4. HAVING >= 3

---

### Problem 9: Customer Retention (Difficult)

**Problem:** Find customers who made purchases in consecutive months.

**Sample Data:**
```
orders:
customer_id | order_date
1           | 2023-01-15
1           | 2023-02-10
1           | 2023-03-05
2           | 2023-01-20
2           | 2023-03-15  -- Gap, not consecutive
```

**Solution:**
```sql
WITH monthly_orders AS (
    SELECT 
        customer_id,
        DATE_TRUNC('month', order_date)::date as month,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY DATE_TRUNC('month', order_date)) as rn
    FROM orders
    GROUP BY customer_id, DATE_TRUNC('month', order_date)
),
month_diff AS (
    SELECT 
        customer_id,
        month,
        DATE_SUB(month, INTERVAL (rn - 1) MONTH) as group_month
    FROM monthly_orders
)
SELECT customer_id
FROM month_diff
GROUP BY customer_id, group_month
HAVING COUNT(*) >= 2;
```

**This is complex - interview approach:**
```
"I would explain the problem step by step:
1. First, get unique months per customer
2. Use ROW_NUMBER to find consecutive months
3. Subtract rn from month to group consecutive ones
4. Count groups with >= 2 months"
```

---

### Problem 10: Sales Rank with Tie Breaking

**Problem:** Rank sales by amount, with tie-breaking by date (earlier = higher rank).

**Solution:**
```sql
SELECT 
    name,
    amount,
    sale_date,
    RANK() OVER (ORDER BY amount DESC, sale_date ASC) as rank
FROM sales;
```

**Key Point:** Multiple ORDER BY columns in OVER clause.

---

## How to Answer Interview Questions

### Template for Interview Success

**When interviewer asks a query problem:**

#### Step 1: Clarify the Problem (1-2 minutes)
```
"Before I start, let me clarify:
- What's the exact output format you want?
- Are there any edge cases? (NULLs, duplicates, etc.)
- What's the data volume? (Affects performance approach)
- Any specific performance requirements?"
```

#### Step 2: Explain Your Approach (1-2 minutes)
```
"My approach:
1. [Step 1]: [Explanation]
2. [Step 2]: [Explanation]
3. [Step 3]: [Explanation]

This handles [edge case] because [reason]"
```

#### Step 3: Write the Query (3-5 minutes)
```sql
-- Write clean, commented code
-- Use CTEs for complex logic
-- Use meaningful aliases
```

#### Step 4: Trace Through Example (1-2 minutes)
```
"Let me trace through with sample data:
- Input: [Sample]
- After step 1: [Result]
- After step 2: [Result]
- Final: [Result]"
```

#### Step 5: Discuss Optimization (1-2 minutes)
```
"Performance considerations:
- Indexes: I'd add index on [column]
- Alternative: Could use [alternative approach]
- Trade-offs: [This approach] is clearer but [slower/faster]"
```

---

## Common Interview Mistakes to Avoid

### ❌ Mistake 1: Using GROUP BY Incorrectly

```sql
-- WRONG: Including non-grouped column
SELECT name, salary, department
FROM employees
GROUP BY department;  -- name not grouped, unpredictable result

-- CORRECT:
SELECT department, AVG(salary)
FROM employees
GROUP BY department;
```

### ❌ Mistake 2: Forgetting WHERE vs HAVING

```sql
-- WRONG:
SELECT department, AVG(salary)
FROM employees
GROUP BY department
WHERE AVG(salary) > 50000;  -- Can't use aggregate in WHERE

-- CORRECT:
SELECT department, AVG(salary)
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;
```

### ❌ Mistake 3: Wrong JOIN Type

```sql
-- WRONG: Using INNER when you need LEFT
SELECT e.name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id;
-- Misses employees with no department

-- CORRECT:
SELECT e.name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;
```

### ❌ Mistake 4: Not Handling NULLs

```sql
-- WRONG:
SELECT salary FROM employees WHERE salary > 50000;
-- If nullable, misses intent

-- BETTER:
SELECT salary FROM employees 
WHERE salary IS NOT NULL AND salary > 50000;
```

### ❌ Mistake 5: Performance - Missing Index Awareness

```sql
-- GOOD: Mention it
"I'd add an index on department_id for JOIN performance"

-- Great: Suggest it specifically
"CREATE INDEX idx_emp_dept ON employees(department_id)"
```

---

## Interview Red Flags & Green Flags

### 🟢 GREEN FLAGS (What interviewers want to see)

1. **Clear communication** - Explain your thinking
2. **Correct syntax** - Knows SQL well
3. **Edge case thinking** - Asks about NULLs, duplicates
4. **Performance awareness** - Mentions indexes, alternatives
5. **Code clarity** - Uses CTEs, meaningful names, comments
6. **Testing mindset** - Traces through examples

### 🔴 RED FLAGS (Avoid these)

1. **Silent coding** - Don't think out loud
2. **Syntax errors** - Use correct syntax
3. **Ignores edge cases** - Ask clarifying questions
4. **Over-complicated** - Prefer simple solutions
5. **No optimization** - Always discuss trade-offs
6. **Wrong approach** - Think before coding

---

## Interview Cheat Sheet

| Problem Type | Approach | Example |
|---|---|---|
| **Ranking** | RANK/DENSE_RANK/ROW_NUMBER | Top N per group |
| **Duplicates** | GROUP BY + HAVING | Find duplicates |
| **Consecutive** | ROW_NUMBER - date | Consecutive days |
| **Self Join** | JOIN table to itself | Employee-Manager |
| **Multi-table** | JOIN + GROUP BY | Aggregate by dept |
| **Complex** | CTE first | Build step-by-step |

---

## Next Steps
→ Move to **Interview Communication Tips**
