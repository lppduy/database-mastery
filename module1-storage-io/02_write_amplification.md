# M1-02: Write Amplification

**Topic:** Storage & I/O Pain — indexes slow down writes

## Situation

Write-heavy app (flash sale, high traffic). After adding indexes last sprint, insert throughput dropped significantly. DB CPU is higher than expected even though reads are fast. Nobody connected the dots between the new indexes and the slowdown.

## Environment

```bash
# PostgreSQL 16 via Docker
docker run --name pglab -e POSTGRES_PASSWORD=secret -p 5432:5432 -d postgres:16

# Connect
docker exec -it pglab psql -U postgres
```

## Theory

When you insert a row, PostgreSQL doesn't just write to the heap file — it also updates every index on that table.

For each `INSERT`:
1. Write the row to the heap page
2. Find the right position in each B-Tree index
3. Insert the index entry, potentially causing **page splits** (when an index page is full, it splits into two)

3 indexes = 4 writes per insert minimum. Under high volume, this adds up fast — **write amplification**.

The more indexes you have, the more expensive every write becomes. Direct tradeoff: reads get faster, writes get slower.

## Simulation

```sql
-- Baseline: no indexes
TRUNCATE orders;

\timing on

INSERT INTO orders (user_id, status, created_at)
SELECT
  (random() * 1000000)::INT,
  (ARRAY['pending','paid','cancelled'])[ceil(random()*3)],
  NOW() - (random() * INTERVAL '365 days')
FROM generate_series(1, 1000000);
-- Time: 1349 ms

-- Now add 3 indexes
TRUNCATE orders;

CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_created_at ON orders(created_at);

-- Same insert
INSERT INTO orders (user_id, status, created_at)
SELECT
  (random() * 1000000)::INT,
  (ARRAY['pending','paid','cancelled'])[ceil(random()*3)],
  NOW() - (random() * INTERVAL '365 days')
FROM generate_series(1, 1000000);
-- Time: 5045 ms
```

## Result

| | No Indexes | 3 Indexes |
|---|---|---|
| Insert 1M rows | 1,349 ms | 5,045 ms |
| Slowdown | — | 3.7x slower |

## The Real Production Trap

Nobody adds 3 indexes in one day — they accumulate over months. Each felt justified at the time. Then write throughput quietly degrades and nobody connects it back to the indexes.

## Takeaway

Every index is a tax on writes, paid forever, to make reads cheaper.

Before adding an index:
- Is this query actually slow in production?
- How write-heavy is this table?
- Can one composite index cover multiple query patterns instead of separate ones?

Audit unused indexes with:
```sql
SELECT * FROM pg_stat_user_indexes WHERE idx_scan = 0;
```
