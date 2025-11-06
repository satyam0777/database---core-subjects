# 🗄️ DBMS Quick Reference Cheat Sheet

**Bookmark this! Review 30 minutes before your interview.**

---

## 1. DBMS Overview

### What is DBMS?
A **Database Management System** is software that:
- Stores data efficiently
- Retrieves data quickly
- Ensures data consistency
- Handles multiple users
- Recovers from failures

### Core Functions
```
Data Definition (DDL)    → CREATE, ALTER, DROP
Data Retrieval (DQL)     → SELECT
Data Manipulation (DML)  → INSERT, UPDATE, DELETE
Data Control (DCL)       → GRANT, REVOKE
Transaction Control (TCL)→ COMMIT, ROLLBACK
```

---

## 2. ACID Properties

| Property | Meaning | Example |
|----------|---------|---------|
| **A**tomicity | All-or-nothing | Transfer $100: deduct from A AND add to B, or neither |
| **C**onsistency | Valid state to valid state | Account total never changes during transfer |
| **I**solation | Transactions don't interfere | Transfer A→B doesn't affect Transfer C→D |
| **D**urability | Committed = permanent | After COMMIT, even if crash, data survives |

### ACID Violation Example

❌ **Without ACID:**
```
Transfer $100 A→B
Deducted from A (A = $900)
CRASH! ⚠️
System recovers
B never gets money (B still $0)
Result: Money lost!
```

✅ **With ACID:**
```
START TRANSACTION
  Deduct from A
  Add to B
COMMIT  ← Both or nothing!
Even if crash, both happen or both don't
Result: Consistency maintained!
```

---

## 3. Transactions

### Transaction States

```
┌─────────┐
│  START  │
└────┬────┘
     │ BEGIN TRANSACTION
     ▼
┌─────────────────┐
│  ACTIVE/RUNNING │◄────────┐
└────┬────────────┘         │
     │                      │ ROLLBACK
     │ (operations...)      │
     ▼                      │
┌─────────────────┐         │
│    PARTIALLY    │─────────┘
│   COMMITTED     │
└────┬────────────┘
     │ COMMIT
     ▼
┌─────────────────┐
│    COMMITTED    │ ← Permanent!
└─────────────────┘
```

### Log-Based Recovery

```
TRANSACTION LOG:
T1 START
T1: UPDATE A=100
T1: INSERT B=50
T1 COMMIT  ← Committed, durable!

T2 START
T2: DELETE C=200
CRASH ⚠️

Recovery:
- T1: Already committed, keep it
- T2: Not committed, roll it back
```

---

## 4. Locking & Concurrency Control

### Lock Types

| Lock Type | Multiple Readers? | Writers? | Usage |
|-----------|-------------------|----------|-------|
| **Shared (S)** | ✅ Yes | ❌ No | READ |
| **Exclusive (X)** | ❌ No | ❌ No | WRITE |

### Timeline Example

```
Time  User A         User B         Resource
  1   LOCK(A, S)   
        Read A=100
  2             LOCK(B, X)
                  Write B=50
  3   LOCK(B, X)
        BLOCKED!
        (waiting for User B)
  4             UNLOCK(B)
  5   LOCK acquired
        Write B=50
```

### Isolation Levels

| Level | Dirty Read | Non-Repeatable | Phantom | Performance |
|-------|-----------|----------------|---------|-------------|
| READ UNCOMMITTED | ✅ Yes | ✅ Yes | ✅ Yes | 🚀 Fastest |
| READ COMMITTED | ❌ No | ✅ Yes | ✅ Yes | 🟡 Medium |
| REPEATABLE READ | ❌ No | ❌ No | ✅ Yes | 🟡 Medium |
| SERIALIZABLE | ❌ No | ❌ No | ❌ No | 🐢 Slowest |

---

## 5. Indexing (B-Tree)

### Why Indexing?

```
❌ WITHOUT INDEX:
  Search 1,000,000 rows one by one
  Time: 1,000,000 comparisons (SLOW!)

✅ WITH B-TREE INDEX:
  Log₂(1,000,000) = 20 comparisons
  Time: 20 comparisons (1000x faster!)
```

### B-Tree Structure

```
B-Tree (Order 3):
          ┌──[30]──┐
         /          \
      [10, 20]    [40, 50, 60]
      / |  \      /  |  |  \
     5 15 25    35 45 55 65

Search 55:
1. Is 55 >= 30? YES → go right
2. Is 55 >= 40? YES → go right
3. Is 55 >= 60? NO → go left
4. Found! (3 comparisons vs millions)
```

### B-Tree vs Hash Index

| Feature | B-Tree | Hash |
|---------|--------|------|
| Range queries | ✅ Yes | ❌ No |
| Ordered | ✅ Yes | ❌ No |
| Equality | ✅ Yes | ✅ Yes |
| Speed (point) | 🟡 O(log n) | ✅ O(1) |
| Space | 🟡 Moderate | ✅ Minimal |

**Use B-Tree for:** Range queries, sorting, most queries
**Use Hash for:** Exact match, speed priority

---

## 6. Query Execution

### Execution Phases

```
1. PARSING
   ├─ Check syntax
   └─ Validate table/column names

2. VALIDATION
   ├─ Check permissions
   └─ Verify data types

3. OPTIMIZATION
   ├─ Generate execution plans
   └─ Choose best one (lowest cost)

4. COMPILATION
   └─ Convert to executable form

5. EXECUTION
   └─ Run the compiled code

6. FETCHING
   └─ Return results to user
```

### Query Plan Example

```sql
SELECT * FROM users WHERE age > 30 AND city = 'NYC'
```

**Bad Plan:**
```
Table Scan (1M rows)
  → Filter age > 30 (300k rows)
  → Filter city = 'NYC' (5k rows)
Time: Scan all 1M rows (SLOW!)
```

**Good Plan:**
```
Index on (city, age)
  → Direct access (B-tree)
  → Get 5k rows matching 'NYC' and age > 30
Time: Few B-tree traversals (FAST!)
```

---

## 7. Storage & Buffer Management

### Disk vs Memory Performance

```
Memory Access:  100 ns   (nanoseconds)
SSD Access:     100 µs   (microseconds)  1000x slower!
HDD Access:     10 ms    (milliseconds)  100,000x slower!

1 disk I/O ≈ 100,000 memory accesses!
```

### Buffer Management

```
When you execute: SELECT * FROM users WHERE id = 5

1. Check Buffer: Is page with id=5 in memory?
   ✅ YES → Use it (fast!)
   ❌ NO → Continue

2. Read from Disk → Load page to buffer
3. Find row 5 in page
4. Return to user

BUFFER HIT RATE = % of accesses already in memory
- 95% hit rate = Usually in memory (FAST!)
- 50% hit rate = Mixed performance
- 10% hit rate = Often reading disk (SLOW!)
```

---

## 8. Deadlock

### What is Deadlock?

```
T1: Lock(A)    T2: Lock(B)
    ↓             ↓
    WAITING FOR B WAITING FOR A
    (Deadlock! Neither can proceed)
```

### Timeline

```
Time  T1 (Transfer A→B)    T2 (Transfer B→A)
  1   LOCK A
  2                         LOCK B
  3   LOCK B (wait...)
  4                         LOCK A (wait...)
  
DEADLOCK! Both waiting forever!
```

### Deadlock Detection & Recovery

```
Detection: Build Wait-For Graph
    T1 → T2 → T1 (cycle = deadlock!)

Recovery: Kill T2 (chosen victim)
    T2 releases locks
    T1 proceeds
    T2 retries later
```

---

## 9. Normalization Forms

### 1NF (First Normal Form)
- Atomic values only
- No repeating groups

```
❌ BAD:
ID | Name    | Hobbies
1  | Alice   | Reading, Gaming, Coding

✅ GOOD:
ID | Name    | Hobby
1  | Alice   | Reading
1  | Alice   | Gaming
1  | Alice   | Coding
```

### 2NF (Second Normal Form)
- 1NF + No partial dependencies

```
❌ BAD:
StudentID | CourseID | StudentName | Grade

(StudentName depends on StudentID, not on key)

✅ GOOD:
Student(StudentID, StudentName)
Enrollment(StudentID, CourseID, Grade)
```

### 3NF (Third Normal Form)
- 2NF + No transitive dependencies

```
❌ BAD:
StudentID | Name    | Department | DeptHead

(DeptHead depends on Department, not key)

✅ GOOD:
Student(StudentID, Name, DeptID)
Department(DeptID, DeptHead)
```

---

## 10. CAP Theorem (Distributed Databases)

**Choose 2 of 3:**

```
       ┌─── C (Consistency) ───┐
       │                       │
       │   All nodes same      │
       │   data at same time   │
       │                       │
   ┌───┴───┐           ┌───────┴────┐
   │       │           │            │
   │       │           │            │
   A       │           │            P
(Availability)      (Partition Tolerance)
   │       │           │            │
   │ All   │    Choose │ Network    │
   │ nodes │      2    │ failures   │
   │ up    │      of   │ handled    │
   │ time  │      3    │            │
   │       │           │            │
   └───┬───┘           └───────┬────┘
       │                       │
       └─────────────────────┘
```

### Examples

| Type | Consistency | Availability | Partition Tolerance |
|------|-------------|--------------|-------------------|
| **PostgreSQL** | ✅ | ✅ | ❌ (single node) |
| **MongoDB** | ✅ | ✅ | ❌ (eventual consistency) |
| **Cassandra** | ❌ | ✅ | ✅ |

---

## 11. Recovery Techniques

### REDO Recovery

```
Before Crash:
WRITE(A, 100)      → In buffer
WRITE(B, 50)       → In buffer
COMMIT             → In log
CRASH! ⚠️

After Recovery:
Read log: "COMMIT for A=100, B=50"
REDO operations → A=100, B=50 restored
```

### UNDO Recovery

```
Before Crash:
WRITE(A, 100)      → In buffer
WRITE(B, 50)       → In buffer
CRASH (before COMMIT)! ⚠️

After Recovery:
Read log: "No COMMIT found"
UNDO operations → A and B unchanged
```

---

## 12. Common Interview Questions - Quick Answers

### Q1: What is ACID?
```
A (Atomicity): All or nothing ✓
C (Consistency): Always valid ✓
I (Isolation): No interference ✓
D (Durability): Permanent after commit ✓
```

### Q2: B-Tree vs Hash Index?
```
B-Tree: Range queries, sorting, ordered ✓
Hash: Exact match only, faster ✓
```

### Q3: What is Deadlock?
```
Two transactions waiting for each other's locks
T1 waits for T2's lock
T2 waits for T1's lock
Neither can proceed
```

### Q4: Explain Locking
```
Shared lock (S): Multiple readers, no writers
Exclusive lock (X): Single writer, no readers
Prevents dirty reads and lost updates
```

### Q5: What is 3NF?
```
Each non-key attribute depends on:
- The whole key (2NF)
- Only the key, nothing else (3NF)
No transitive dependencies
```

### Q6: Query Optimization Steps?
```
1. Parse query
2. Generate multiple execution plans
3. Estimate cost for each plan
4. Choose lowest cost plan
5. Execute it
```

### Q7: Buffer Management?
```
Check if page in buffer memory
If yes: Use it (fast - memory access)
If no: Load from disk (slow - disk I/O)
Minimize disk I/O = Maximize performance
```

### Q8: Recovery from Crash?
```
Check transaction log
COMMITTED transactions: REDO them
UNCOMMITTED transactions: UNDO them
Result: Consistent state restored
```

---

## 13. Performance Red Flags 🚨

### What Causes Slow Queries?

```
❌ Full table scan (no index)
❌ N+1 queries (query in loop)
❌ Unnecessary joins
❌ Wrong index type
❌ Poor query plan
❌ Too many locks (contention)
❌ Disk I/O bottleneck
```

### Optimization Checklist

```
✅ Add indexes on WHERE, JOIN, ORDER BY columns
✅ Use EXPLAIN to see query plan
✅ Reduce columns selected (SELECT *)
✅ Batch operations (reduce disk I/O)
✅ Use appropriate isolation level
✅ Monitor buffer hit ratio
✅ Cache frequently accessed data
```

---

## 14. Interview Favorites - Last Minute Review

**Most Asked:**
1. ACID properties (ALWAYS asked!)
2. Indexing (B-tree, why indexes matter)
3. Transactions & locking
4. Query optimization
5. Normalization
6. Recovery & durability
7. Concurrency control
8. Deadlock

**Must Know Cold:**
- ACID with examples
- B-tree structure
- Locking timeline
- Recovery process
- Isolation levels
- JOIN algorithms

---

## 15. Formulas & Rules of Thumb

### B-Tree Search Complexity
```
Average: O(log n)
Best: O(1)
Worst: O(log n)
```

### Transaction States
```
START → ACTIVE → COMMITTED (or ABORTED)
```

### Lock Compatibility

```
         Requests S    Requests X
Holds S    ✅ Yes       ❌ No
Holds X    ❌ No        ❌ No
```

### Normalization Order
```
1NF → 2NF → 3NF → BCNF
(Each stricter than previous)
```

---

## Before Interview: 30-Minute Review

```
10 min: Review ACID properties
5 min:  B-tree structure
5 min:  Locking timeline
5 min:  Query execution phases
3 min:  Recovery process
2 min:  Common questions in section 14
```

---

## What to Say vs NOT Say

### ✅ SAY THIS:
```
"Transactions ensure ACID properties...
Indexes use B-tree structure...
Locking prevents dirty reads...
Query optimizer chooses best plan...
Buffer management minimizes disk I/O..."
```

### ❌ DON'T SAY THIS:
```
"I don't know how databases work internally"
"Indexes just make things faster"
"Locking is confusing"
"Query optimization is magic"
"Durability just happens somehow"
```

---

## Your Interview Toolkit

```
✅ ACID examples ready
✅ B-tree diagram ready to draw
✅ Locking scenarios understood
✅ Recovery process memorized
✅ Isolation levels clear
✅ Normalization forms understood
✅ Query optimization steps ready
✅ Common mistakes known
```

---

**You've got this! 💪**

**Next: Read `01_DBMS_FUNDAMENTALS.md` for deep dive**

*Last updated: November 6, 2025*
