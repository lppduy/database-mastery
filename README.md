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

## Module 6: Interview Prep

Distilled theory after production depth is solid — ACID, CAP, isolation levels, index rules, common questions answered with production context.

---

## Progress

| Module | Status |
|--------|--------|
| 01 Storage & I/O Pain | 3/4 |
| 02 Transactions & Concurrency | - |
| 03 Query Internals | - |
| 04 Schema & Application Patterns | - |
| 05 Scaling | - |
| 06 Interview Prep | - |
