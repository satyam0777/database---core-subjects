# 🐘 PostgreSQL Cheat Sheet - Quick Reference
**One-page lookup for everything you need**

---

## 📋 Connection Strings

```sql
-- Basic connection
psql -U username -d database_name -h localhost

-- With password
psql -U username -d database_name -h localhost -W

-- From within SQL
\c database_name

-- Connection string format
postgresql://username:password@localhost:5432/database_name
```

---

## 🗄️ Database & User Commands

```sql
-- Create database
CREATE DATABASE db_name;

-- Create user
CREATE USER username WITH PASSWORD 'password';

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE db_name TO username;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO username;

-- List databases
\l (in psql)

-- List users
\du (in psql)

-- Drop database
DROP DATABASE IF EXISTS db_name;
```

---

## 📊 Table Operations

```sql
-- Create table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    name TEXT NOT NULL,
    age INT CHECK (age >= 18),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Alter table
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
ALTER TABLE users DROP COLUMN phone;
ALTER TABLE users RENAME TO customers;

-- Drop table
DROP TABLE IF EXISTS users CASCADE;

-- Copy table structure
CREATE TABLE users_backup AS TABLE users WITH NO DATA;

-- Truncate (fast delete all)
TRUNCATE TABLE users;
```

---

## 🔑 Constraints

```sql
-- PRIMARY KEY
id SERIAL PRIMARY KEY

-- UNIQUE
email VARCHAR(255) UNIQUE

-- NOT NULL
name VARCHAR(100) NOT NULL

-- FOREIGN KEY
user_id INT REFERENCES users(id)

-- CHECK
age INT CHECK (age >= 0)

-- DEFAULT
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP

-- Composite Primary Key
PRIMARY KEY (user_id, order_id)

-- Add constraint
ALTER TABLE users ADD CONSTRAINT email_unique UNIQUE(email);

-- Drop constraint
ALTER TABLE users DROP CONSTRAINT email_unique;
```

---

## 📝 Data Types

| Type | Example | Use Case |
|------|---------|----------|
| INT | `12345` | Whole numbers |
| BIGINT | `9223372036854775807` | Very large numbers |
| DECIMAL(10,2) | `123.45` | Money/precise decimals |
| FLOAT | `123.45` | Scientific numbers |
| VARCHAR(n) | `'text'` | Variable length text |
| TEXT | `'long text'` | Long text (unlimited) |
| CHAR(n) | `'fixed'` | Fixed length text |
| BOOLEAN | `true/false` | True or false |
| TIMESTAMP | `2025-11-06 10:00:00` | Date and time |
| DATE | `2025-11-06` | Date only |
| TIME | `10:00:00` | Time only |
| UUID | `550e8400-e29b-41d4-a716-446655440000` | Unique ID |
| JSONB | `{"key": "value"}` | JSON data (indexed) |
| ARRAY | `ARRAY[1,2,3]` | Arrays of values |
| BYTEA | `\\x1234` | Binary data |

---

## 🔍 Indexes

```sql
-- Create index
CREATE INDEX idx_email ON users(email);

-- Create unique index
CREATE UNIQUE INDEX idx_username ON users(username);

-- Create composite index
CREATE INDEX idx_user_created ON orders(user_id, created_at);

-- Drop index
DROP INDEX idx_email;

-- List indexes
\di (in psql)
SELECT * FROM pg_indexes WHERE tablename = 'users';

-- Analyze query plan
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- Verbose analysis (with actual time)
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';
```

---

## 🔄 Transactions

```sql
-- Start transaction
BEGIN;
-- or
START TRANSACTION;

-- Commit
COMMIT;

-- Rollback
ROLLBACK;

-- Savepoint
SAVEPOINT sp1;
ROLLBACK TO sp1;

-- Set isolation level
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Isolation levels:
-- READ UNCOMMITTED (dirty reads possible)
-- READ COMMITTED (default)
-- REPEATABLE READ
-- SERIALIZABLE (strictest)
```

---

## 📤 Data Manipulation

```sql
-- INSERT
INSERT INTO users (email, name) VALUES ('test@example.com', 'John');

-- INSERT multiple
INSERT INTO users (email, name) VALUES 
    ('test1@example.com', 'John'),
    ('test2@example.com', 'Jane');

-- UPDATE
UPDATE users SET name = 'Jane' WHERE email = 'test@example.com';

-- DELETE
DELETE FROM users WHERE id = 1;

-- RETURNING (PostgreSQL specific!)
INSERT INTO users (email) VALUES ('new@example.com') RETURNING id, email;
UPDATE users SET name = 'John' WHERE id = 1 RETURNING *;
DELETE FROM users WHERE id = 1 RETURNING *;
```

---

## 🔗 JOINs

```sql
-- INNER JOIN
SELECT u.name, o.id FROM users u 
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN
SELECT u.name, o.id FROM users u 
LEFT JOIN orders o ON u.id = o.user_id;

-- RIGHT JOIN
SELECT u.name, o.id FROM users u 
RIGHT JOIN orders o ON u.id = o.user_id;

-- FULL OUTER JOIN
SELECT u.name, o.id FROM users u 
FULL OUTER JOIN orders o ON u.id = o.user_id;

-- CROSS JOIN (cartesian product)
SELECT u.name, p.name FROM users u CROSS JOIN products p;

-- Self join
SELECT u1.name, u2.name FROM users u1 
JOIN users u2 ON u1.manager_id = u2.id;
```

---

## 🎯 Aggregates & GROUP BY

```sql
-- COUNT
SELECT COUNT(*) FROM users;
SELECT COUNT(DISTINCT email) FROM users;

-- SUM, AVG, MIN, MAX
SELECT SUM(amount), AVG(amount) FROM orders;
SELECT MIN(price), MAX(price) FROM products;

-- GROUP BY
SELECT user_id, COUNT(*) FROM orders GROUP BY user_id;
SELECT user_id, SUM(amount) FROM orders GROUP BY user_id;

-- HAVING (filter after GROUP BY)
SELECT user_id, COUNT(*) as order_count FROM orders 
GROUP BY user_id HAVING COUNT(*) > 5;

-- Window functions
SELECT user_id, amount, 
       ROW_NUMBER() OVER (ORDER BY amount DESC),
       RANK() OVER (PARTITION BY user_id ORDER BY amount DESC)
FROM orders;
```

---

## 📊 Useful Queries

```sql
-- Last N records
SELECT * FROM users ORDER BY created_at DESC LIMIT 10;

-- Pagination
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 20;

-- Distinct values
SELECT DISTINCT city FROM customers;

-- Check NULL
SELECT * FROM users WHERE email IS NULL;
SELECT * FROM users WHERE email IS NOT NULL;

-- String operations
SELECT * FROM users WHERE name LIKE 'John%';
SELECT * FROM users WHERE name ILIKE 'john%'; -- case insensitive
SELECT * FROM users WHERE name ~ '^John'; -- regex

-- Date operations
SELECT * FROM orders WHERE created_at > NOW() - INTERVAL '7 days';
SELECT * FROM orders WHERE DATE(created_at) = '2025-11-06';

-- CAST
SELECT CAST(id AS TEXT) FROM users;
SELECT id::TEXT FROM users; -- shorter syntax

-- CASE statement
SELECT id, name, 
       CASE WHEN age < 18 THEN 'Minor' 
            WHEN age >= 18 THEN 'Adult' 
            ELSE 'Unknown' 
       END as age_group
FROM users;
```

---

## 🔐 Performance Tips

```sql
-- 1. Always use indexes on WHERE clauses
CREATE INDEX idx_email ON users(email);

-- 2. Avoid SELECT *
SELECT id, name FROM users;  -- Better

-- 3. Use LIMIT for large results
SELECT * FROM users LIMIT 1000;

-- 4. Batch inserts
INSERT INTO users (email, name) VALUES (...), (...), (...);

-- 5. VACUUM (reclaim space)
VACUUM users;
VACUUM ANALYZE users; -- also update statistics

-- 6. Check slow queries
SET log_min_duration_statement = 1000; -- log queries > 1s

-- 7. Use EXPLAIN to debug
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';

-- 8. Denormalize if needed
-- Add frequently accessed data to parent table

-- 9. Archive old data
-- Move old records to archive table

-- 10. Connection pooling
-- Use PgBouncer for production
```

---

## 👥 Users & Permissions

```sql
-- Create user
CREATE USER app_user WITH PASSWORD 'secure_password';

-- Grant privileges
GRANT CONNECT ON DATABASE mydb TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;
GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA public TO app_user;

-- Grant specific table
GRANT SELECT ON users TO app_user;
GRANT INSERT, UPDATE ON orders TO app_user;

-- Revoke privileges
REVOKE INSERT ON orders FROM app_user;

-- Alter user password
ALTER USER app_user WITH PASSWORD 'new_password';

-- Drop user
DROP USER IF EXISTS app_user;

-- List user privileges
\dp (in psql)
SELECT * FROM information_schema.role_table_grants;
```

---

## 📋 Useful System Queries

```sql
-- List all tables
SELECT tablename FROM pg_tables WHERE schemaname = 'public';

-- List columns in table
SELECT column_name, data_type FROM information_schema.columns 
WHERE table_name = 'users';

-- Get table size
SELECT pg_size_pretty(pg_total_relation_size('users'));

-- Get database size
SELECT pg_size_pretty(pg_database_size('mydb'));

-- Check index usage
SELECT schemaname, tablename, indexname FROM pg_indexes 
WHERE tablename = 'users';

-- Unused indexes
SELECT * FROM pg_stat_user_indexes WHERE idx_scan = 0;

-- Check table statistics
SELECT * FROM pg_stat_user_tables WHERE relname = 'users';

-- Check active connections
SELECT * FROM pg_stat_activity;

-- Kill connection
SELECT pg_terminate_backend(pid) FROM pg_stat_activity 
WHERE pid <> pg_backend_pid();
```

---

## 🔧 Configuration

```sql
-- Check configuration
SHOW max_connections;
SHOW shared_buffers;
SHOW work_mem;

-- Set configuration (session)
SET work_mem = '256MB';
SET synchronous_commit = OFF;

-- View all configuration
SELECT * FROM pg_settings;

-- postgresql.conf location
-- Linux: /var/lib/postgresql/data/postgresql.conf
-- Mac: /usr/local/var/postgres/postgresql.conf
-- Windows: C:\Program Files\PostgreSQL\data\postgresql.conf

-- Performance settings (modify postgresql.conf)
shared_buffers = 256MB
effective_cache_size = 1GB
work_mem = 16MB
checkpoint_completion_target = 0.9
```

---

## 🆘 Troubleshooting

```sql
-- Connection refused
-- Check: postgres is running, port 5432, credentials

-- Slow queries
EXPLAIN ANALYZE SELECT ...;
-- Look for Seq Scan (bad), use indexes

-- Deadlock
-- Review transaction order
-- Use timeouts: SET lock_timeout = '5s';

-- Disk full
SELECT pg_database_size('dbname');
-- Move data or add storage

-- Too many connections
-- Check: max_connections setting
-- Use connection pooling (PgBouncer)

-- Missing indexes
-- Use pg_stat_user_tables to find sequential scans
-- Create indexes on filtered columns

-- Bloated tables
VACUUM FULL users;  -- careful, locks table!
CLUSTER users USING idx_primary;  -- rebuilds table

-- Check table integrity
REINDEX TABLE users;
```

---

## 📚 Common PostgreSQL Features

```sql
-- RETURNING clause (get inserted data)
INSERT INTO users (name) VALUES ('John') RETURNING id, name;

-- UPSERT (INSERT ... ON CONFLICT)
INSERT INTO users (email, name) VALUES ('test@example.com', 'John')
ON CONFLICT (email) DO UPDATE SET name = 'John Updated';

-- Common Table Expressions (CTE / WITH)
WITH recent_orders AS (
    SELECT * FROM orders WHERE created_at > NOW() - INTERVAL '7 days'
)
SELECT user_id, COUNT(*) FROM recent_orders GROUP BY user_id;

-- JSONB operations
SELECT data->>'email' FROM users;  -- get JSON value
SELECT * FROM users WHERE data @> '{"status": "active"}';

-- ARRAY operations
SELECT * FROM users WHERE skills @> ARRAY['Python', 'SQL'];
SELECT * FROM users WHERE 'Python' = ANY(skills);

-- Full-text search
SELECT * FROM articles WHERE content @@ 'PostgreSQL & Django';

-- Date functions
SELECT NOW();
SELECT CURRENT_DATE;
SELECT CURRENT_TIMESTAMP;
SELECT DATE_TRUNC('month', created_at) FROM users;

-- String functions
SELECT UPPER(name) FROM users;
SELECT LOWER(email) FROM users;
SELECT CONCAT(first_name, ' ', last_name) FROM users;
SELECT LENGTH(name) FROM users;
SELECT SUBSTRING(name FROM 1 FOR 3) FROM users;

-- Math functions
SELECT ABS(-5);
SELECT ROUND(3.7);
SELECT CEIL(3.2);
SELECT FLOOR(3.7);
```

---

## 🎯 Quick Tips

```
1. Always use SERIAL or UUID for IDs
2. Use TIMESTAMP with timezone awareness
3. Use TEXT for unlimited strings, VARCHAR(n) for limits
4. Use JSONB for flexible schemas (indexed!)
5. Index WHERE clauses, not SELECT
6. Use transactions for consistency
7. Regular VACUUM keeps tables healthy
8. Monitor with pg_stat_*
9. Use EXPLAIN ANALYZE for debugging
10. Connection pooling in production (PgBouncer)
11. Foreign keys maintain referential integrity
12. Constraints are your friend (CHECK, UNIQUE, etc)
13. Use RETURNING to reduce round trips
14. UPSERT instead of DELETE + INSERT
15. Backup regularly!
```

---

## 📞 Common Errors

```
ERROR: relation "users" does not exist
→ Table doesn't exist, check spelling

ERROR: duplicate key value violates unique constraint
→ Duplicate insert, use ON CONFLICT DO UPDATE

ERROR: foreign key constraint is violated
→ Referenced record doesn't exist

ERROR: column "x" does not exist
→ Column name typo, check table schema

FATAL: too many connections
→ Increase max_connections or use connection pooling

ERROR: permission denied for schema public
→ Need GRANT USAGE ON SCHEMA public

ERROR: database is locked
→ VACUUM FULL is running, wait or kill it
```

---

## 🔗 Useful psql Commands

```
\l           - List databases
\dt          - List tables
\du          - List users
\dp          - List permissions
\di          - List indexes
\d tablename - Describe table
\h SQL       - Help on SQL command
\?           - List psql commands
\q           - Quit
\copy        - Copy data
\x           - Toggle expanded output
\timing      - Toggle query timing
```

---

**Use this during interview for quick mental reference!**  
**Master the top commands and you'll ace PostgreSQL questions.**

🚀 Ready? Open `01_POSTGRESQL_FUNDAMENTALS.md` next!
