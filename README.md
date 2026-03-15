# Database Mastery

Scenario-driven. Hit a real production problem → understand why → learn the theory → simulate it → fix it.

**Stack:** PostgreSQL + Docker, data generation in Java or Python.

---

## Philosophy

> You hit a real production problem → understand why it happens → learn the theory → simulate it → know how to fix it.

## Session Format (per scenario)

1. **Situation** — real production problem
2. **Theory** — why it happens
3. **Simulation** — reproduce it with SQL / Docker
4. **Fix** — apply the solution
5. **Takeaway** — what to remember

---

## Module 1: Storage & I/O Pain

| # | Scenario | Concepts |
|---|----------|----------|
| 01 | Insert 10M rows, query without index → watch it crawl | Heap files, full table scan, pages |
| 02 | Add index, re-run → measure the difference | B-Tree internals, index scan vs seq scan |
| 03 | Composite index, wrong column order → index ignored | Selectivity, prefix rules |
| 04 | High write load + index → writes slow down | Write amplification, index maintenance cost |

## Module 2: Transactions & Concurrency

| # | Scenario | Concepts |
|---|----------|----------|
| 01 | Two sessions update same row simultaneously | Locks, blocking, ACID |
| 02 | Deadlock between two transactions | Lock ordering, detection, prevention |
| 03 | Read stale data mid-transaction | Isolation levels, dirty/phantom/non-repeatable read |
| 04 | High read load without locks | MVCC, how Postgres avoids read locks |

## Module 3: Query Internals

| # | Scenario | Concepts |
|---|----------|----------|
| 01 | Slow JOIN on a large table | Hash join vs nested loop vs merge join |
| 02 | Query planner picks wrong index | Statistics, ANALYZE, cardinality |
| 03 | EXPLAIN ANALYZE a bad query, fix it | Reading query plans end-to-end |

## Module 4: Schema & Application Patterns

| # | Scenario | Concepts |
|---|----------|----------|
| 01 | API returns 1000 users + each user's orders → N+1 | Eager load, query batching |
| 02 | Paginate 10M rows with OFFSET → gets slower | Cursor-based pagination |
| 03 | Delete rows but keep audit trail | Soft delete, partial index |
| 04 | Denormalize for speed, consistency breaks | Normalization tradeoffs |

## Module 5: Scaling

| # | Scenario | Concepts |
|---|----------|----------|
| 01 | Partition a 10M row table, query hits only 1 partition | Range/hash partition, pruning |
| 02 | Read traffic spikes → add read replica | Replication lag, leader/follower |
| 03 | Single table becomes hotspot | Sharding strategy, hotspot problem |

## Module 6: ERD Design & SQL Problem Solving

Real-world schema design + SQL query challenges. Goal: solve almost any SQL problem confidently.

### Part A: ERD Design

| # | Scenario | Concepts |
|---|----------|----------|
| 01 | Design a schema for an e-commerce system | ERD, normalization, 1NF/2NF/3NF, relationships |
| 02 | Design a schema for a chat system | Many-to-many, message threading, self-referencing |
| 03 | Design a schema for a ride-sharing app | Geospatial, status transitions, polymorphic relations |
| 04 | Design a schema for a booking system | Conflict detection, time ranges, seat locking |

### Part B: SQL Fundamentals

| # | Topic | Concepts |
|---|-------|----------|
| 05 | JOINs deep dive | INNER, LEFT, RIGHT, FULL, CROSS, SELF join — when to use each |
| 06 | Aggregation & grouping | GROUP BY, HAVING, COUNT/SUM/AVG/MIN/MAX, NULL traps |
| 07 | Subqueries & CTEs | Correlated subquery, WITH, WITH RECURSIVE |
| 08 | Set operations | UNION, UNION ALL, INTERSECT, EXCEPT |
| 09 | NULL handling | IS NULL, COALESCE, NULLIF, NULL in aggregation |

### Part C: Advanced SQL

| # | Topic | Concepts |
|---|-------|----------|
| 10 | Window functions | ROW_NUMBER, RANK, DENSE_RANK, NTILE |
| 11 | Window frames | LEAD, LAG, FIRST_VALUE, LAST_VALUE, OVER(PARTITION BY ORDER BY) |
| 12 | Running totals & moving averages | SUM OVER, AVG OVER, cumulative patterns |
| 13 | Gaps and islands | Find missing sequences, consecutive groups |
| 14 | Top-N per group | Rank per category, deduplicate, latest per user |
| 15 | Pivot / unpivot | CASE WHEN aggregation, crosstab |

### Part D: Query Optimization

| # | Topic | Concepts |
|---|-------|----------|
| 16 | Rewriting slow queries | Subquery vs JOIN, EXISTS vs IN, avoiding SELECT * |
| 17 | Index-friendly queries | Avoid function on column, SARGable predicates |
| 18 | EXPLAIN ANALYZE in depth | Seq scan vs index scan, cost estimation, row estimates |
| 19 | Pagination at scale | OFFSET trap, cursor-based pagination |
| 20 | Common interview problems | LeetCode-style SQL: nth highest, duplicates, consecutive rows |

## Module 7: Interview Prep

Distilled theory after production depth is solid — ACID, CAP, isolation levels, index rules, common questions answered with production context.

---

## Progress

| Module | Status |
|--------|--------|
| 01 Storage & I/O Pain | ✓ 4/4 done |
| 02 Transactions & Concurrency | - |
| 03 Query Internals | - |
| 04 Schema & Application Patterns | - |
| 05 Scaling | - |
| 06 ERD Design & SQL Problem Solving | 1/20 in progress |
| 07 Interview Prep | - |
