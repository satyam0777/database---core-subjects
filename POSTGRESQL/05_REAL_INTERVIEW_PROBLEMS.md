# 15+ Real PostgreSQL Interview Problems
**With complete solutions, follow-ups, and what interviewers look for**

---

## 📌 How to Use This File

Each problem has:
- **Problem:** What interviewer asks
- **Time:** How long to answer
- **What they look for:** Evaluation criteria
- **Model Answer:** Complete solution
- **Follow-ups:** Common next questions
- **Red Flags:** Mistakes to avoid
- **Pro Tip:** Stand out technique

---

## PROBLEM 1: Design a Users Table
**Difficulty:** ⭐ Easy | **Time:** 90 seconds

### **Problem**
```
"Design a users table for a social media application. 
What columns would you include? What constraints?"
```

### **What They're Looking For**
- ✅ Proper data types
- ✅ Relevant constraints (PRIMARY KEY, UNIQUE, NOT NULL)
- ✅ Good column names
- ✅ Thinking about validation
- ❌ Over-engineering
- ❌ Forgetting constraints

### **Model Answer**

```sql
CREATE TABLE users (
    -- Primary identification
    id SERIAL PRIMARY KEY,  -- Auto-incrementing unique ID
    
    -- Essential information
    email VARCHAR(255) UNIQUE NOT NULL,  -- Unique email, always needed
    username VARCHAR(50) UNIQUE NOT NULL,  -- Unique username
    
    -- Password and security
    password_hash VARCHAR(255) NOT NULL,  -- Never store plain password!
    
    -- User profile
    first_name VARCHAR(100),  -- Optional
    last_name VARCHAR(100),   -- Optional
    bio TEXT,  -- Longer text for bio
    avatar_url VARCHAR(500),  -- URL to avatar image
    
    -- Account status
    is_active BOOLEAN DEFAULT true,  -- Can deactivate account
    
    -- Metadata
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,  -- When created
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,  -- When updated
    last_login_at TIMESTAMP  -- For activity tracking
);

-- Add index on frequently queried columns
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);
```

### **Explanation**

```
Column choices:

1. ID (SERIAL PRIMARY KEY)
   - Every user needs unique identifier
   - SERIAL auto-generates (1, 2, 3, ...)
   - PRIMARY KEY enforces uniqueness

2. EMAIL and USERNAME (UNIQUE NOT NULL)
   - Both are login identifiers
   - UNIQUE prevents duplicates
   - NOT NULL ensures always present
   - Both indexed for fast lookup

3. PASSWORD_HASH (NOT NULL)
   - NEVER store plain passwords!
   - Always hash using bcrypt/argon2
   - This is what's stored

4. TIMESTAMPS
   - created_at: Audit trail, analytics
   - updated_at: Track changes
   - last_login_at: Activity tracking

5. INDEXES
   - email and username indexed
   - Speed up login queries
```

### **Follow-up Questions**

**Q1: "Should phone be here?"**
```
A: "It depends on requirements. If login via phone, add it with UNIQUE. 
   If just optional contact, can add later or in separate table."
```

**Q2: "What about roles/permissions?"**
```
A: "Good question! In a real system, I'd have separate tables:
   - roles (admin, moderator, user)
   - user_roles (many-to-many relationship)
   
   This is more flexible than a role column."
```

**Q3: "How handle soft deletes?"**
```
A: "Add deleted_at TIMESTAMP. Instead of DELETE, set deleted_at = NOW().
   Then WHERE deleted_at IS NULL in queries. Preserves data."
```

### **Red Flags**

❌ "I'd add a 'name' column without first and last"
   - Better to split for searching flexibility

❌ "Store password in plaintext"
   - Security disaster - never do this!

❌ "Use email as PRIMARY KEY"
   - Email can change, ID shouldn't

❌ "No timestamps"
   - Hard to debug, poor analytics

### **Pro Tips**

✅ Add `updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP` and trigger
```sql
CREATE TRIGGER update_users_timestamp 
BEFORE UPDATE ON users 
FOR EACH ROW 
EXECUTE FUNCTION update_timestamp();
```

✅ Consider partitioning for millions of users

✅ Archive old deleted accounts separately

---

## PROBLEM 2: Write Efficient Query with Indexes
**Difficulty:** ⭐⭐ Medium | **Time:** 120 seconds

### **Problem**
```
"I have a users table with 10 million records. 
Users complain the search is slow. How would you debug and fix?"
```

### **What They're Looking For**
- ✅ EXPLAIN ANALYZE knowledge
- ✅ Understanding query optimization
- ✅ Index creation
- ✅ Problem-solving approach
- ✅ Metrics/measurement
- ❌ Adding random indexes
- ❌ Not measuring first

### **Model Answer**

```
Step 1: Measure the problem
─────────────────────────────

EXPLAIN ANALYZE 
SELECT * FROM users 
WHERE email = 'test@example.com';

Look for "Seq Scan" - means full table scan (bad!)
Look at "Planning Time" and "Execution Time"

Example output:
                        QUERY PLAN
──────────────────────────────────────────────
 Seq Scan on users  (cost=0.00..195000.00 rows=1)
   Filter: (email = 'test@example.com')
 Planning time: 0.123 ms
 Execution time: 456.789 ms ← SLOW!

Step 2: Add Index
────────────────

CREATE INDEX idx_users_email ON users(email);

Step 3: Verify it works
──────────────────────

EXPLAIN ANALYZE 
SELECT * FROM users 
WHERE email = 'test@example.com';

Should now show "Index Scan":
                           QUERY PLAN
───────────────────────────────────────────────
 Index Scan using idx_users_email on users 
   Index Cond: (email = 'test@example.com')
 Planning time: 0.145 ms
 Execution time: 0.234 ms ← 2000x FASTER!

Step 4: Analyze Index
──────────────────────

ANALYZE users;  -- Update statistics

Step 5: Monitor
───────────────

SELECT * FROM pg_stat_user_indexes 
WHERE tablename = 'users';

Check idx_scan count - should increase with queries
```

### **Complete Solution Code**

```sql
-- 1. Find what queries are slow
SELECT query, calls, mean_exec_time 
FROM pg_stat_statements 
WHERE query LIKE '%users%' 
ORDER BY mean_exec_time DESC;

-- 2. For each slow query, use EXPLAIN
EXPLAIN ANALYZE
SELECT * FROM users 
WHERE email = 'test@example.com' 
AND is_active = true;

-- 3. Add appropriate indexes
CREATE INDEX idx_users_email_active 
ON users(email, is_active) 
WHERE is_active = true;  -- Partial index!

-- 4. Verify performance
EXPLAIN ANALYZE
SELECT * FROM users 
WHERE email = 'test@example.com' 
AND is_active = true;

-- 5. Check index usage
SELECT schemaname, tablename, indexname, idx_scan 
FROM pg_stat_user_indexes 
WHERE tablename = 'users';

-- 6. If still slow, check for missing statistics
ANALYZE users;
VACUUM users;

-- 7. Monitor query performance
SELECT query, calls, mean_exec_time 
FROM pg_stat_statements 
WHERE query LIKE '%users%' 
ORDER BY mean_exec_time DESC;
```

### **Key Concepts**

```
Sequential Scan (bad):
├─ Reads every single row
├─ Time: O(n) - grows with table size
└─ Only used if no good index exists

Index Scan (good):
├─ Uses index to find rows
├─ Time: O(log n) - much faster
└─ Should see this for WHERE clauses

Why EXPLAIN ANALYZE:
├─ EXPLAIN shows plan estimate
├─ ANALYZE actually runs query
├─ Compares plan vs reality
└─ Best for debugging

Partial Indexes:
├─ Only index rows matching WHERE
├─ Smaller, faster index
├─ Great for soft deletes: WHERE deleted_at IS NULL
└─ Perfect for is_active = true
```

### **Follow-up Questions**

**Q: "When would an index NOT help?"**
```
A: 
1. Scanning most of the table (no filtering)
2. Very small table (< 10k rows)
3. Column has low cardinality (few unique values)
4. Column with mostly NULL values
5. Column rarely used in WHERE clause
```

**Q: "How many indexes should I create?"**
```
A: 
- Rule of thumb: 3-5 indexes per table usually enough
- Don't over-index (slows down INSERT/UPDATE)
- Index columns in WHERE clauses
- Monitor unused indexes:

SELECT * FROM pg_stat_user_indexes 
WHERE idx_scan = 0;  -- Never used!
```

### **Red Flags**

❌ "Just add index to every column"
   - Slows down writes, wastes space

❌ "Don't know EXPLAIN exists"
   - EXPLAIN ANALYZE is essential skill

❌ "SELECT * with no WHERE"
   - Can't help with indexes, needs different fix

### **Pro Tips**

✅ Use composite indexes for common WHERE patterns:
```sql
-- If often: WHERE email = x AND is_active = true
CREATE INDEX idx_email_active ON users(email, is_active);
```

✅ Use partial indexes to reduce size:
```sql
CREATE INDEX idx_active_users ON users(email) 
WHERE is_active = true;
```

---

## PROBLEM 3: Handle Concurrent Updates
**Difficulty:** ⭐⭐⭐ Hard | **Time:** 150 seconds

### **Problem**
```
"Two users try to transfer money simultaneously. 
How ensure money doesn't disappear or duplicate?"
```

### **What They're Looking For**
- ✅ Transaction understanding
- ✅ ACID knowledge
- ✅ Locking concepts
- ✅ Real-world scenario
- ✅ Error handling
- ❌ No consideration of concurrency
- ❌ Race conditions

### **Model Answer**

```sql
-- THE PROBLEM (Race condition without transactions)
────────────────────────────────────────────────

Time | User A                  | User B
──────────────────────────────────────────────────
t1   | READ balance = $100    |
t2   |                         | READ balance = $100
t3   | WRITE balance = $50    |
t4   |                         | WRITE balance = $50

Result: Only $50 deducted instead of $100!

-- THE SOLUTION (With transactions and locks)
────────────────────────────────────────────────

BEGIN;
  -- Lock the row for update (no one else can modify)
  SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;
  
  -- Check balance
  IF balance >= 100 THEN
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
  ELSE
    ROLLBACK;  -- Insufficient funds
  END IF;
  
COMMIT;
```

### **Complete Transaction Example**

```sql
-- Function to safely transfer money
CREATE FUNCTION transfer_money(
  from_account_id INT,
  to_account_id INT,
  amount DECIMAL
) RETURNS TEXT AS $$
BEGIN
  -- Start transaction
  BEGIN
    -- Lock both rows in consistent order (prevents deadlock)
    -- Order: always lock lower ID first
    IF from_account_id < to_account_id THEN
      SELECT balance FROM accounts WHERE id = from_account_id FOR UPDATE;
      SELECT balance FROM accounts WHERE id = to_account_id FOR UPDATE;
    ELSE
      SELECT balance FROM accounts WHERE id = to_account_id FOR UPDATE;
      SELECT balance FROM accounts WHERE id = from_account_id FOR UPDATE;
    END IF;
    
    -- Check balance
    IF (SELECT balance FROM accounts WHERE id = from_account_id) < amount THEN
      RETURN 'Insufficient funds';
    END IF;
    
    -- Transfer
    UPDATE accounts 
    SET balance = balance - amount, 
        updated_at = CURRENT_TIMESTAMP
    WHERE id = from_account_id;
    
    UPDATE accounts 
    SET balance = balance + amount,
        updated_at = CURRENT_TIMESTAMP
    WHERE id = to_account_id;
    
    RETURN 'Transfer successful';
  EXCEPTION WHEN OTHERS THEN
    RETURN 'Transfer failed: ' || SQLERRM;
  END;
END;
$$ LANGUAGE plpgsql;

-- Usage
SELECT transfer_money(1, 2, 100);
```

### **Locking Strategies**

```
1. Optimistic Locking (version field)
   ─────────────────────────────────
   
   CREATE TABLE accounts (
     id INT PRIMARY KEY,
     balance DECIMAL,
     version INT DEFAULT 0
   );
   
   UPDATE accounts 
   SET balance = balance - 100, version = version + 1
   WHERE id = 1 AND version = 5;  -- Only if version unchanged
   
   Pros: Better concurrency
   Cons: Must handle failures

2. Pessimistic Locking (row locks)
   ───────────────────────────────
   
   SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
   UPDATE accounts SET balance = balance - 100 WHERE id = 1;
   
   Pros: Guaranteed consistency
   Cons: Lower concurrency

3. Isolation Levels
   ───────────────
   
   READ UNCOMMITTED  - Highest concurrency, lowest consistency (don't use!)
   READ COMMITTED    - Default, good for most cases
   REPEATABLE READ   - More consistent
   SERIALIZABLE      - Highest consistency, lowest concurrency
```

### **Follow-up Questions**

**Q: "What's a deadlock?"**
```
A: "When two transactions lock resources in opposite order:

Time | Transaction A      | Transaction B
───────────────────────────────────────
t1   | LOCK account 1    |
t2   |                    | LOCK account 2
t3   | WAIT FOR account 2 | WAIT FOR account 1
t4   | DEADLOCK! Both waiting

Solution: Always lock in same order (lowest ID first)"
```

### **Red Flags**

❌ "Just use SELECT then UPDATE without transaction"
   - Race condition! Other transaction might change data

❌ "Don't lock rows"
   - Money can disappear!

❌ "Lock in different order"
   - Can cause deadlocks

---

## PROBLEM 4: Optimize N+1 Query Problem
**Difficulty:** ⭐⭐ Medium | **Time:** 120 seconds

### **Problem**
```
"List all users with their posts. Why is this slow? Fix it."
```

### **The Problem Code**

```javascript
// WRONG - N+1 Query Problem
────────────────────────────

const users = await db.query('SELECT * FROM users');

for (const user of users) {
  // THIS RUNS ONCE PER USER! 10,000 users = 10,001 queries!
  user.posts = await db.query(
    'SELECT * FROM posts WHERE user_id = $1',
    [user.id]
  );
}

Database queries:
1. SELECT * FROM users          (1 query)
2. SELECT * FROM posts WHERE user_id = 1  (1 query per user)
3. SELECT * FROM posts WHERE user_id = 2
...
Result: 10,001 queries for 10,000 users! SLOW!
```

### **Solution 1: Using JOIN**

```sql
-- Use JOIN to get everything in one query
SELECT 
  u.id, u.email, u.name,
  p.id as post_id, p.title, p.content
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
ORDER BY u.id;

Result:
user_id | email           | post_id | title
────────────────────────────────────────
1       | john@example    | 100     | First post
1       | john@example    | 101     | Second post
2       | jane@example    | 200     | Jane's post

Database queries: 1 (much better!)

Downside: Need to merge results in code
```

### **Solution 2: Using Batch Queries**

```sql
-- Get all users
SELECT * FROM users;

-- Get all posts for those users in ONE query
SELECT * FROM posts WHERE user_id IN (1, 2, 3, ..., 10000);

Database queries: 2 (better!)

JavaScript:
const users = await db.query('SELECT * FROM users');
const userIds = users.map(u => u.id);

const posts = await db.query(
  'SELECT * FROM posts WHERE user_id = ANY($1)',
  [userIds]
);

// Merge in code
const postsMap = new Map();
posts.forEach(p => {
  if (!postsMap.has(p.user_id)) {
    postsMap.set(p.user_id, []);
  }
  postsMap.get(p.user_id).push(p);
});

users.forEach(u => {
  u.posts = postsMap.get(u.id) || [];
});
```

### **Solution 3: Using Subquery (Best for many-to-one)**

```sql
SELECT 
  u.*,
  (SELECT json_agg(json_build_object(
    'id', p.id,
    'title', p.title,
    'content', p.content
  )) FROM posts p WHERE p.user_id = u.id)
  as posts
FROM users u;

Result JSON:
{
  "id": 1,
  "email": "john@example.com",
  "posts": [
    {"id": 100, "title": "First post"},
    {"id": 101, "title": "Second post"}
  ]
}

Database queries: 1
Returns nested JSON structure
```

### **Why N+1 is Bad**

```
N users × M posts = N+1 queries

10,000 users × 5 posts each:

WRONG (N+1):
  1 query to get users
  + 10,000 queries for posts
  = 10,001 queries ❌

RIGHT (JOIN):
  1 query ✅

Performance difference:
  Wrong: 50 seconds
  Right: 0.5 seconds
  = 100x faster!
```

### **Follow-up Questions**

**Q: "But what about 1000 users with 1000 items each?"**
```
A: "Good point. JOIN creates cartesian product.

Options:
1. Still use JOIN - 1 million rows but 1 query
2. Paginate - get 100 users at a time
3. Use subquery for nested JSON
4. Cache results

Choose based on needs."
```

---

## PROBLEM 5: Design Database for 1M Users
**Difficulty:** ⭐⭐⭐ Hard | **Time:** 180 seconds

### **Problem**
```
"Design a database for 1 million users and 100 million posts. 
What optimizations would you make?"
```

### **Schema Design**

```sql
-- USERS TABLE (1M records)
CREATE TABLE users (
    id BIGINT PRIMARY KEY,  -- BIGINT for large scale
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(50) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Indexes for common queries
    CONSTRAINT idx_email ON (email),
    CONSTRAINT idx_username ON (username)
);

-- POSTS TABLE (100M records)
-- Problem: Too big to have all indexes!
CREATE TABLE posts (
    id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES users(id),
    title VARCHAR(255) NOT NULL,
    content TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Only essential index
    CONSTRAINT idx_user_posts ON (user_id, created_at)
);

-- PARTITION for performance
-- Split 100M posts into monthly partitions
CREATE TABLE posts_2025_11 PARTITION OF posts
  FOR VALUES FROM ('2025-11-01') TO ('2025-12-01');

CREATE TABLE posts_2025_12 PARTITION OF posts
  FOR VALUES FROM ('2025-12-01') TO ('2026-01-01');
```

### **Optimization Strategies**

```
1. INDEXING STRATEGY
   ─────────────────
   
   Don't index everything (too slow to write):
   ✓ Index user_id on posts (for lookups)
   ✓ Index created_at on posts (for sorting)
   ✗ Don't index content (too big)
   
   Use partial indexes:
   CREATE INDEX idx_active_posts ON posts(user_id)
   WHERE deleted_at IS NULL;

2. PARTITIONING
   ────────────
   
   Split posts table by date:
   - posts_2025_01 (Jan posts)
   - posts_2025_02 (Feb posts)
   - posts_2025_03 (Mar posts)
   
   Benefits:
   - Smaller indexes per partition
   - Can delete old partitions
   - Parallel queries across partitions

3. DENORMALIZATION
   ───────────────
   
   Add summary data to users table:
   ALTER TABLE users ADD COLUMN post_count INT DEFAULT 0;
   
   Update with trigger:
   CREATE TRIGGER increment_post_count
   AFTER INSERT ON posts
   FOR EACH ROW
   EXECUTE FUNCTION increment_user_post_count();
   
   Why:
   - Fast COUNT(*) queries
   - No need to count 100M rows

4. CACHING
   ───────
   
   Cache with Redis:
   - Popular posts: posts:trending:7days
   - User profiles: user:123
   - Post feeds: user:123:feed
   
   PostgreSQL stores everything,
   Redis caches hot data.

5. READ REPLICAS
   ──────────────
   
   Read-heavy workload:
   - Primary: handles writes (users creating posts)
   - Replica 1: handles read queries (viewing posts)
   - Replica 2: handles analytics
   
   Distribution:
   - Writes → Primary
   - Reads → Any replica
```

### **Query Optimization**

```sql
-- BAD: Counts all 100M records
SELECT COUNT(*) FROM posts;

-- BETTER: Use denormalized column
SELECT post_count FROM users WHERE id = 123;

-- BAD: Full table scan
SELECT * FROM posts WHERE content LIKE '%searchterm%';

-- BETTER: Use full-text search
CREATE INDEX idx_content_search ON posts USING GIN(to_tsvector('english', content));
SELECT * FROM posts WHERE to_tsvector('english', content) @@ 'searchterm';

-- BAD: Join everything
SELECT u.*, (SELECT COUNT(*) FROM posts WHERE user_id = u.id)
FROM users u;

-- BETTER: Pre-calculated count
SELECT u.*, u.post_count FROM users u;

-- BAD: Get all posts
SELECT * FROM posts ORDER BY created_at DESC LIMIT 10;

-- BETTER: Paginate
SELECT * FROM posts 
WHERE created_at < (SELECT created_at FROM posts OFFSET 100 LIMIT 1)
ORDER BY created_at DESC 
LIMIT 10;
```

---

## PROBLEM 6: Backup and Recovery Strategy
**Difficulty:** ⭐⭐ Medium | **Time:** 90 seconds

### **Problem**
```
"Design a backup strategy for a production PostgreSQL database. 
How ensure data can be recovered?"
```

### **Complete Strategy**

```
BACKUP LAYERS:

1. DAILY FULL BACKUPS (retain 30 days)
   ──────────────────────────────────
   
   pg_basebackup -D /backups/daily/$(date +%Y%m%d) -Ft -z
   
   Size: ~100GB for 1TB database
   Time: ~30 minutes
   Storage: 3TB for 30 days

2. HOURLY INCREMENTAL (WAL archiving)
   ──────────────────────────────────
   
   Enable in postgresql.conf:
   wal_level = replica
   archive_mode = on
   archive_command = 'cp %p /backups/wal/%f'
   
   Size: ~50MB per hour
   Enables point-in-time recovery

3. CROSS-REGION REPLICATION
   ───────────────────────────
   
   Primary (US-East)
        ↓ Streaming replication
   Standby (US-West)
        ↓ Continuous backup
   S3 (AWS)
   
   Survives data center failure

4. AUTOMATED TESTING
   ──────────────────
   
   Weekly: Restore backup to test database
   - Verify data integrity
   - Test recovery time
   - Document any issues
```

### **Recovery Scenarios**

```sql
-- SCENARIO 1: Table corrupted, need to restore
──────────────────────────────────────────────

-- Stop application traffic
SELECT pg_ctl('stop', 'fast');

-- Restore full backup
pg_restore /backups/daily/20251106 -d myapp

-- Recover up to specific time
recovery_target_timeline = 'latest'
recovery_target_time = '2025-11-06 10:30:00'

-- Then restart
SELECT pg_ctl('start');

Recovery time: 10-30 minutes (depending on size)


-- SCENARIO 2: Accidental DELETE (recovered from WAL)
────────────────────────────────────────────────────

-- Use point-in-time recovery
recovery_target_time = '2025-11-06 10:29:00'  -- Before delete

Recovery time: 5-10 minutes


-- SCENARIO 3: Server crashed, use standby
───────────────────────────────────────────

-- Promote standby to primary
SELECT pg_promote();

-- Application now connects to new primary
-- Data loss: None (continuous replication)
-- Downtime: 30 seconds
```

### **Monitoring**

```sql
-- Check backup age
SELECT 
  (CURRENT_TIMESTAMP - pg_postmaster_start_time()) as backup_age,
  pg_size_pretty(pg_database_size('myapp')) as size;

-- Monitor replication
SELECT 
  pid, usename, state, sync_state, 
  write_lag, flush_lag, replay_lag
FROM pg_stat_replication;

-- Check WAL archiving
SELECT * FROM pg_stat_archiver;
```

---

## PROBLEM 7: Query Optimization - Complex Query
**Difficulty:** ⭐⭐⭐ Hard | **Time:** 120 seconds

### **Problem**
```
"Write a query to get top 10 users by post count in last 30 days 
with engagement metrics. Optimize it."
```

### **Solution**

```sql
-- NAIVE APPROACH (Slow)
────────────────────────

SELECT 
  u.id, u.username, 
  COUNT(p.id) as post_count,
  COUNT(c.id) as comment_count,
  COUNT(l.id) as like_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id AND p.created_at > NOW() - INTERVAL '30 days'
LEFT JOIN comments c ON p.id = c.post_id
LEFT JOIN likes l ON p.id = l.post_id
GROUP BY u.id
ORDER BY post_count DESC
LIMIT 10;

Problems:
- Multiple LEFT JOINs cause multiplication (1 post × 100 comments × 50 likes = 5000 rows!)
- Counts wrong due to cartesian product
- Multiple passes needed


-- OPTIMIZED APPROACH (Fast)
─────────────────────────────

WITH user_stats AS (
  -- Get post counts per user
  SELECT 
    user_id,
    COUNT(*) as post_count
  FROM posts
  WHERE created_at > NOW() - INTERVAL '30 days'
  GROUP BY user_id
),
engagement_stats AS (
  -- Get engagement per post
  SELECT 
    p.user_id,
    COUNT(DISTINCT c.id) as comment_count,
    COUNT(DISTINCT l.id) as like_count
  FROM posts p
  LEFT JOIN comments c ON p.id = c.post_id
  LEFT JOIN likes l ON p.id = l.post_id
  WHERE p.created_at > NOW() - INTERVAL '30 days'
  GROUP BY p.user_id
)
SELECT 
  u.id, u.username,
  COALESCE(us.post_count, 0) as post_count,
  COALESCE(es.comment_count, 0) as comment_count,
  COALESCE(es.like_count, 0) as like_count
FROM users u
LEFT JOIN user_stats us ON u.id = us.user_id
LEFT JOIN engagement_stats es ON u.id = es.user_id
ORDER BY post_count DESC
LIMIT 10;

Why faster:
- Uses CTEs to break into logical steps
- Each CTE processes data independently
- No cartesian product
- PostgreSQL optimizer can parallelize
- 10-100x faster for large datasets


-- CREATE INDEXES
──────────────

CREATE INDEX idx_posts_created ON posts(created_at);
CREATE INDEX idx_posts_user ON posts(user_id);
CREATE INDEX idx_comments_post ON comments(post_id);
CREATE INDEX idx_likes_post ON likes(post_id);

-- VERIFY WITH EXPLAIN
──────────────────────

EXPLAIN ANALYZE <query>;

Look for:
✓ Index Scan (good)
✓ Sequential Scan on small tables (OK)
✗ Sequential Scan on millions of rows (bad - needs index)
```

---

## PROBLEM 8: Data Migration Safely
**Difficulty:** ⭐⭐⭐ Hard | **Time:** 150 seconds

### **Problem**
```
"Need to rename email column to email_address with no downtime. 
How do this safely?"
```

### **Solution: Zero-Downtime Migration**

```sql
-- STEP 1: Create new column
──────────────────────────

ALTER TABLE users ADD COLUMN email_address VARCHAR(255);

-- Application still uses old 'email' column


-- STEP 2: Copy data (can take time, no lock)
─────────────────────────────────────────

UPDATE users SET email_address = email;

-- Monitor progress
SELECT COUNT(*) FROM users WHERE email_address IS NULL;


-- STEP 3: Create unique constraint
──────────────────────────────────

CREATE UNIQUE INDEX idx_email_address ON users(email_address);


-- STEP 4: Deploy application to use NEW column
───────────────────────────────────────────────

-- Application now reads/writes email_address
-- Both columns exist


-- STEP 5: Verify data integrity
──────────────────────────────

-- Check no NULL values
SELECT COUNT(*) FROM users WHERE email_address IS NULL;

-- Check no duplicates
SELECT email_address, COUNT(*) FROM users 
GROUP BY email_address 
HAVING COUNT(*) > 1;


-- STEP 6: Add constraint for new writes
──────────────────────────────────────

ALTER TABLE users 
ADD CONSTRAINT email_address_not_null CHECK (email_address IS NOT NULL);


-- STEP 7: Drop old column (when confident)
───────────────────────────────────────

ALTER TABLE users DROP COLUMN email CASCADE;

-- Rename if desired
ALTER TABLE users RENAME COLUMN email_address TO email;


-- STEP 8: Rename index
──────────────────────

ALTER INDEX idx_email_address RENAME TO idx_email;

Timeline:
┌─ Deploy step 1: Add column
├─ Run step 2-3: Copy data (happens in background)
├─ Deploy step 4: Switch to new column
├─ Verify: Check data integrity
├─ Deploy step 5-6: Add constraints
├─ Wait: Make sure everything stable (hours/days)
└─ Deploy step 7-8: Drop old column

KEY: No downtime, can rollback at any point
```

---

## PROBLEM 9: Connection Pooling in Production
**Difficulty:** ⭐⭐ Medium | **Time:** 90 seconds

### **Problem**
```
"Application connects to PostgreSQL. With 100 concurrent users,
connection count goes to 1000+. How fix?"
```

### **Solution: PgBouncer Connection Pooling**

```
WITHOUT POOLING:
───────────────
100 users × 10 connections each = 1000 connections
PostgreSQL limit: 200 connections (default)
Result: Connection refused errors

WITH POOLING:
─────────────
100 users → PgBouncer → 20 connections to PostgreSQL
PgBouncer queues extra requests
PostgreSQL stays at 20 connections
Result: Works smoothly!


PGBOUNCER SETUP:
────────────────

1. Install pgbouncer
   sudo apt install pgbouncer

2. Configure /etc/pgbouncer/pgbouncer.ini
   
   [databases]
   myapp = host=localhost port=5432 dbname=myapp
   
   [pgbouncer]
   pool_mode = transaction  ; or session
   max_client_conn = 500    ; max from app
   default_pool_size = 25   ; connections to DB
   min_pool_size = 5
   reserve_pool_size = 5
   reserve_pool_timeout = 3

3. Start pgbouncer
   systemctl start pgbouncer

4. Update application connection string
   OLD: postgresql://user:pass@localhost:5432/myapp
   NEW: postgresql://user:pass@localhost:6432/myapp
                                                ↑
                                        PgBouncer port


POOL MODES:
──────────

1. Session (one client = one connection)
   - Used when: Client maintains connection
   - Pro: Simple
   - Con: More connections needed

2. Transaction (one client = connection per transaction)
   - Used when: Client opens/closes per query
   - Pro: Few connections needed
   - Con: Can't use prepared statements across trans

3. Statement (one client = connection per statement)
   - Used when: Maximum sharing needed
   - Pro: Minimum connections needed
   - Con: No multi-statement transactions
```

---

## PROBLEM 10: Deadlock Prevention
**Difficulty:** ⭐⭐⭐ Hard | **Time:** 120 seconds

### **Problem**
```
"Two concurrent transactions deadlock. How prevent?"
```

### **Solution**

```sql
DEADLOCK SCENARIO:
─────────────────

Transaction A:                  Transaction B:
BEGIN;                          BEGIN;
LOCK account 1 ✓               
                                LOCK account 2 ✓
WAIT for account 2 ⏳          WAIT for account 1 ⏳
  DEADLOCK!                       DEADLOCK!
Both transactions killed ❌


FIX 1: LOCK IN SAME ORDER
────────────────────────

Transfer function:
1. If from_id < to_id: lock from first
2. If from_id > to_id: still lock lower ID first

CREATE FUNCTION transfer_money(from_id INT, to_id INT) AS $$
BEGIN
  IF from_id < to_id THEN
    SELECT * FROM accounts WHERE id = from_id FOR UPDATE;
    SELECT * FROM accounts WHERE id = to_id FOR UPDATE;
  ELSE
    SELECT * FROM accounts WHERE id = to_id FOR UPDATE;
    SELECT * FROM accounts WHERE id = from_id FOR UPDATE;
  END IF;
  
  -- Safe to update now
  UPDATE accounts SET balance = balance - 100 WHERE id = from_id;
  UPDATE accounts SET balance = balance + 100 WHERE id = to_id;
END;
$$ LANGUAGE plpgsql;


FIX 2: USE HIGHER ISOLATION LEVEL
──────────────────────────────────

SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

Serializable mode prevents conflicts automatically.
Downside: Lower concurrency.


FIX 3: RETRY LOGIC
──────────────────

Python example:
import psycopg2
from psycopg2.errors import TransactionRollbackError

max_retries = 3
for attempt in range(max_retries):
    try:
        conn.execute(transfer_query)
        conn.commit()
        break
    except TransactionRollbackError:
        if attempt < max_retries - 1:
            time.sleep(0.1 * (2 ** attempt))  # Exponential backoff
        else:
            raise


FIX 4: TIMEOUT
──────────────

SET lock_timeout = '5s';  -- Kill locks after 5 seconds

Or in postgresql.conf:
lock_timeout = '5s'
statement_timeout = '30s'


MONITORING DEADLOCKS:
─────────────────────

-- Find deadlocks in log
grep "deadlock detected" /var/log/postgresql/postgresql.log

-- Current blocking queries
SELECT pid, usename, query, state 
FROM pg_stat_activity 
WHERE state != 'idle';

-- Kill blocking query
SELECT pg_terminate_backend(pid);
```

---

## PROBLEM 11: JSON/JSONB Usage
**Difficulty:** ⭐⭐ Medium | **Time:** 90 seconds

### **Problem**
```
"Store flexible user preferences (theme, language, notifications, etc).
When use JSONB vs separate columns?"
```

### **Solution**

```sql
-- Create table with JSONB
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255),
    
    -- Store flexible preferences as JSONB
    preferences JSONB DEFAULT '{}'::jsonb
);

-- Insert data
INSERT INTO users (email, preferences) VALUES (
    'john@example.com',
    '{
        "theme": "dark",
        "language": "en",
        "notifications": {
            "email": true,
            "sms": false,
            "push": true
        },
        "privacy": "public",
        "timezone": "UTC"
    }'::jsonb
);

-- Query JSONB
SELECT preferences->>'theme' FROM users WHERE id = 1;
-- Returns: "dark"

SELECT preferences->'notifications'->>'email' FROM users WHERE id = 1;
-- Returns: true

-- Filter by JSONB
SELECT * FROM users WHERE preferences->>'theme' = 'dark';

-- Check if contains key
SELECT * FROM users WHERE preferences ? 'language';

-- Update JSONB
UPDATE users SET preferences = jsonb_set(preferences, '{theme}', '"light"')
WHERE id = 1;

-- Index JSONB for performance
CREATE INDEX idx_preferences_theme ON users USING GIN(preferences);

-- Search within JSONB
SELECT * FROM users WHERE preferences @> '{"theme": "dark"}';
-- Returns: All users with dark theme


WHEN USE JSONB:
────────────

✓ Store flexible attributes
✓ Schema evolves frequently
✓ Attributes differ per user
✓ Need semi-structured data

Examples:
- User preferences
- Event metadata
- API responses
- Configuration settings


WHEN USE SEPARATE COLUMNS:
──────────────────────────

✓ Fixed schema
✓ Need frequent filtering
✓ Performance critical
✓ All users have field

Examples:
- Email, name, created_at
- Required fields
- High-cardinality data


JSONB PERFORMANCE TIPS:
──────────────────────

1. Index with GIN for containment queries
   CREATE INDEX idx_prefs ON users USING GIN(preferences);

2. Index specific fields for filtering
   CREATE INDEX idx_theme ON users((preferences->>'theme'));

3. Use ->> for text (can use indexes)
   SELECT * FROM users WHERE preferences->>'theme' = 'dark';

4. Use @> for containment (slower but flexible)
   SELECT * FROM users WHERE preferences @> '{"notifications":{"email":true}}';
```

---

## PROBLEM 12-15: Quick Fire Questions

### **Problem 12: Difference between VARCHAR and TEXT**
```
VARCHAR(n) - max length n characters
TEXT - unlimited characters

Recommendation:
- Use TEXT (simpler, same performance)
- Or VARCHAR with reasonable limit (100, 255)
- PostgreSQL treats both similarly

Answer: "VARCHAR is fine if you know max length. 
TEXT is more flexible. Performance is same."
```

### **Problem 13: When would you denormalize?**
```
Normalize: Separate tables, relationships
Denormalize: Add redundant data for performance

Example:
Normalized:
  users (id, name)
  orders (id, user_id, amount)
  
Denormalized:
  orders (id, user_id, user_name, amount)
  - Added user_name even though it's in users table
  
Why: Count(orders) gives total orders. Don't need join.
Cost: Must update both tables when name changes.

Use when: Read-heavy, queries are slow, write-performance okay
```

### **Problem 14: Explain full-text search**
```
PostgreSQL has native full-text search:

CREATE INDEX idx_search ON articles USING GIN(to_tsvector('english', content));

SELECT * FROM articles 
WHERE to_tsvector('english', content) @@ 'PostgreSQL & database';

Returns all articles matching terms: PostgreSQL OR database
Handles: stemming, stop words, rankings

Better than: LIKE '%search%' (slow, no features)
Worse than: Elasticsearch (but simpler)
```

### **Problem 15: How handle million-row batch insert?**
```
DON'T do 1 million individual INSERTs.

DO this:
1. Bulk insert with COPY
   COPY users(email, name) FROM stdin;
   
2. Or batch inserts
   INSERT INTO users VALUES 
     (...), (...), (...), ... (10k rows per batch)
   
3. Disable indexes during insert
   ALTER TABLE users DISABLE TRIGGER ALL;
   -- bulk insert
   ALTER TABLE users ENABLE TRIGGER ALL;
   REINDEX TABLE users;

Result: 1 million inserts in < 1 second
```

---

## 🎯 Summary: Interview Strategies

### **For Each Problem:**

1. **Clarify Requirements** (30 seconds)
   - Ask about data size
   - Ask about read/write ratio
   - Ask about constraints

2. **Explain Approach** (30 seconds)
   - Outline solution high-level
   - Mention tradeoffs
   - Ask if they want details

3. **Show Code** (60 seconds)
   - Write actual SQL
   - Explain key parts
   - Mention optimizations

4. **Handle Follow-ups** (remaining time)
   - Answer questions
   - Discuss alternatives
   - Show you know alternatives

---

## 🎉 Ready for PostgreSQL Interviews!

You now know 15+ real problems with complete solutions.

**Next Step:** Practice writing these from memory!

Record yourself explaining each one.

**Target:** Can answer without looking at notes.

---

**Section Status:** ✅ Complete  
**Problems Covered:** 15+ real scenarios  
**Interview Readiness:** 70%+ (with practice!)
