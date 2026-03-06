# M1-04: Unused Indexes

**Topic:** Storage & I/O Pain — paying write tax with zero read benefit

## Situation

Your team has been adding indexes for months — every slow query got one. Write throughput has been creeping down but nobody knows why. You audit the database and discover several indexes that no query ever uses. You're paying the write amplification cost on every insert, update, and delete — getting zero read benefit in return.

## Environment

```bash
# PostgreSQL 16 via Docker
docker run --name pglab -e POSTGRES_PASSWORD=secret -p 5432:5432 -d postgres:16

# Connect
docker exec -it pglab psql -U postgres
```

## Theory

Every index is maintained automatically by PostgreSQL on every write — regardless of whether any query uses it. Over time, indexes accumulate from:
- One-off investigations ("let me add this and see")
- Old features removed but index left behind
- Redundant indexes (e.g. `(user_id)` becomes useless when `(user_id, status)` exists)

PostgreSQL tracks usage in `pg_stat_user_indexes`. If `idx_scan = 0`, no query has used that index since the last stats reset.

---

## EXPLAIN vs EXPLAIN ANALYZE

These are the two tools you'll use constantly to diagnose query performance.

```sql
EXPLAIN SELECT * FROM orders WHERE user_id = 42;
```
- Shows the **query plan only** — what PostgreSQL *plans* to do
- Does **not execute** the query
- Costs are estimates (in arbitrary "work units", not ms)
- Fast and safe to run on production anytime

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 42;
```
- **Actually executes** the query, then shows the plan + real timings
- Shows both estimated cost and actual time/rows
- Reveals when the planner's estimates are wrong (e.g. estimated 11 rows, actual 7)
- Do NOT run on expensive write queries in production without caution — it executes for real

**Key fields to read:**

| Field | Meaning |
|-------|---------|
| `cost=0.00..120.00` | Estimated cost (startup..total). Not milliseconds |
| `rows=11` | Estimated rows returned |
| `actual time=0.05..0.12` | Real time in ms (startup..total) |
| `rows=7` | Actual rows returned |
| `loops=1` | How many times this node executed |
| `Rows Removed by Filter` | Rows scanned but discarded — high = inefficient |

When estimated rows differ a lot from actual rows, it means PostgreSQL has stale statistics → run `ANALYZE orders` to refresh.

---

## Simulation

```sql
-- Check index usage on the orders table
SELECT
  indexrelname,   -- name of the index
  idx_scan,       -- how many times a query used this index
  idx_tup_read,   -- index entries read across all scans
  idx_tup_fetch   -- actual heap rows fetched using the index
FROM pg_stat_user_indexes
WHERE relname = 'orders'
ORDER BY idx_scan DESC;
```

Example output:
```
  indexrelname   | idx_scan | idx_tup_read | idx_tup_fetch
-----------------+----------+--------------+---------------
 idx_user_status |        1 |            1 |             1
 orders_pkey     |        0 |            0 |             0
 idx_status_user |        0 |            0 |             0
```

### Reading the columns

**`idx_scan`** — scan count. Most important. Zero = unused.

**`idx_tup_read`** — entries read from the index itself. Can be higher than `idx_tup_fetch` because some entries get filtered out before hitting the heap (MVCC visibility checks).

**`idx_tup_fetch`** — actual rows pulled from the table. If much lower than `idx_tup_read`, the index has low selectivity.

**Practical read:**
- High `idx_scan`, high `idx_tup_fetch` → heavily used, keep it
- High `idx_scan`, low `idx_tup_fetch` → used often but not selective, consider redesigning
- `idx_scan = 0` → drop candidate

### How long does it track?

Stats accumulate **since the last reset** — not by week. Check when stats were last reset:

```sql
SELECT stats_reset FROM pg_stat_bgwriter;
```

Rule of thumb: if `idx_scan = 0` after **2-4 weeks of normal production traffic**, it's safe to consider unused.

### Fix: drop the unused index

```sql
DROP INDEX idx_status_user;
```

## Production Audit Query

Run this periodically — sort by `idx_scan ASC`, anything at 0 after weeks of traffic is a drop candidate:

```sql
SELECT
  relname AS table,
  indexrelname AS index,
  idx_scan,
  pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
ORDER BY idx_scan ASC;
```

## Takeaway

Unused indexes are silent performance killers on write-heavy tables. Audit regularly with `pg_stat_user_indexes`. Drop what isn't used — smaller write amplification, less disk, faster inserts.
