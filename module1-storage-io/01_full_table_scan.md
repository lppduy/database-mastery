# M1-01: Full Table Scan

**Topic:** Storage & I/O Pain — no index on a large table

## Situation

Dashboard queries timing out. A query that used to return in 10ms now takes seconds. The table has grown to 10M rows. Nobody added an index.

## Theory

Without an index, PostgreSQL has no choice but to read every page of the table — a **sequential scan (seq scan)**. Data is stored in 8KB pages on disk; 10M rows means thousands of pages to load and check.

With a **B-Tree index**, PostgreSQL builds a sorted tree. Finding a value becomes O(log n) — it traverses the tree in ~23 steps for 10M rows and jumps directly to the matching pages.

Key terms:
- **Heap file** — actual table data, stored in pages, unordered
- **Seq scan** — reads every page, O(n)
- **Index scan** — follows B-Tree, O(log n)
- **cost** in query plans — PostgreSQL's estimated work units, not ms

## Environment

```bash
# PostgreSQL 16 via Docker
docker run --name pglab -e POSTGRES_PASSWORD=secret -p 5432:5432 -d postgres:16

# Connect
docker exec -it pglab psql -U postgres
```

## Simulation

```sql
-- Create table
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INT,
  status VARCHAR(20),
  created_at TIMESTAMP
);

-- Insert 10M rows
INSERT INTO orders (user_id, status, created_at)
SELECT
  (random() * 1000000)::INT,
  (ARRAY['pending','paid','cancelled'])[ceil(random()*3)],
  NOW() - (random() * INTERVAL '365 days')
FROM generate_series(1, 10000000);
```

## Before Index

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 42;
```

```
Gather (cost=1000.00..120219.24 rows=11 width=23) (actual time=71.507..180.123 rows=7 loops=1)
  Workers Planned: 2
  Workers Launched: 2
  -> Parallel Seq Scan on orders
       Filter: (user_id = 42)
       Rows Removed by Filter: 3333331
Execution Time: 180.543 ms
```

PostgreSQL spun up 3 workers, each scanned ~3.3M rows, discarded all but 7.

## Fix

```sql
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

## After Index

```
Index Scan using idx_orders_user_id on orders
  Index Cond: (user_id = 42)
Execution Time: 0.158 ms
```

## Result

| | Seq Scan | Index Scan |
|---|---|---|
| Rows examined | 10,000,000 | 7 |
| Execution time | 180.543 ms | 0.158 ms |
| Speedup | — | ~1,140x faster |

## Takeaway

An unindexed column on a large table turns every lookup into a full scan. As the table grows, it gets linearly worse. The fix is cheap — but knowing *which* columns to index is the real skill.
