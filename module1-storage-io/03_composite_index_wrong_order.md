# M1-03: Composite Index — Wrong Column Order

**Topic:** Storage & I/O Pain — index exists but gets ignored

## Situation

A slow query is reported. There's already an index on the table. The dev verified it worked in testing. But in production, `EXPLAIN ANALYZE` shows a seq scan. The index exists but PostgreSQL isn't using it.

## Environment

```bash
# PostgreSQL 16 via Docker
docker run --name pglab -e POSTGRES_PASSWORD=secret -p 5432:5432 -d postgres:16

# Connect
docker exec -it pglab psql -U postgres
```

## Theory

A composite index on `(status, user_id)` is like a phone book sorted by last name then first name.

- `WHERE status = 'paid'` → uses index (leftmost column)
- `WHERE status = 'paid' AND user_id = 42` → uses index (both columns)
- `WHERE user_id = 42` → index ignored, full scan (skipped the leftmost)

This is the **prefix rule**: PostgreSQL can only use a composite index if the query filters on the leftmost column(s) first.

**Column order rule of thumb:**
1. Equality filters first (`WHERE user_id = ?`)
2. Range filters last (`WHERE created_at > ?`)

## Simulation

```sql
-- Setup: only composite index in wrong order
DROP INDEX IF EXISTS idx_orders_user_id;
CREATE INDEX idx_status_user ON orders(status, user_id);

-- Query filters on user_id only — skips leading column
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 42;
```

```
Parallel Seq Scan on orders
  Filter: (user_id = 42)
  Rows Removed by Filter: 333333
Execution Time: 42.738 ms
```

Index ignored — seq scan.

## Fix

```sql
-- Correct order: user_id first
CREATE INDEX idx_user_status ON orders(user_id, status);

EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 42;
```

```
Index Scan using idx_user_status on orders
  Index Cond: (user_id = 42)
Execution Time: 0.122 ms
```

## Result

| | Wrong order `(status, user_id)` | Correct order `(user_id, status)` |
|---|---|---|
| Query: `WHERE user_id = 42` | Seq scan | Index scan |
| Execution time | 42.738 ms | 0.122 ms |

## Bonus: Covering Index

```sql
-- Both columns in SELECT match the index — no heap access needed
EXPLAIN ANALYZE SELECT user_id, status FROM orders WHERE user_id = 42;
-- → Index Only Scan (answers entirely from the index)
```

## Takeaway

Put the most selective, most frequently filtered column first in a composite index. A query that skips the leading column gets no benefit from the index at all.
