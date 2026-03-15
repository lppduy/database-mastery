# CLAUDE.md

## Context
Database Mastery study session, PostgreSQL + Docker, scenario-driven learning.

## Teaching Style
- Start with a real production situation (the pain)
- Explain the theory behind why it happens
- Simulate it hands-on with SQL and Docker
- Apply the fix
- End with a clear takeaway
- Go one scenario at a time

## Session Format
1. Situation — real production problem
2. Theory — why it happens
3. Simulation — reproduce with SQL / Docker
4. Fix — apply the solution
5. Takeaway — what to remember

## Focus
- Real production depth first
- Interview prep after production understanding is solid

## Stack
- PostgreSQL 16 via Docker (`docker run --name pglab -e POSTGRES_PASSWORD=secret -p 5432:5432 -d postgres:16`)
- Connect: `docker exec -it pglab psql -U postgres`
- Java or Python for data generation

## Notes
- Always include the Docker setup block (image version, run command, connect command) in each scenario file
- After each scenario is fully understood: create the `.md` file, update progress in CLAUDE.md and README.md, commit and push — all together, without waiting for user to ask

## Progress
- [x] M1-01 Full Table Scan
- [x] M1-02 Write Amplification (index + high write load)
- [x] M1-03 Composite Index Wrong Order
- [x] M1-04 Unused Indexes + EXPLAIN vs EXPLAIN ANALYZE

## Module 6: ERD Design & SQL Problem Solving
### Part A: ERD Design
- [x] M6-01 E-commerce schema
- [ ] M6-02 Chat system schema
- [ ] M6-03 Ride-sharing schema
- [ ] M6-04 Booking system schema

### Part B: SQL Fundamentals
- [ ] M6-05 JOINs deep dive
- [ ] M6-06 Aggregation & grouping
- [ ] M6-07 Subqueries & CTEs
- [ ] M6-08 Set operations
- [ ] M6-09 NULL handling

### Part C: Advanced SQL
- [ ] M6-10 Window functions (ROW_NUMBER, RANK, DENSE_RANK)
- [ ] M6-11 Window frames (LEAD, LAG, FIRST/LAST_VALUE)
- [ ] M6-12 Running totals & moving averages
- [ ] M6-13 Gaps and islands
- [ ] M6-14 Top-N per group
- [ ] M6-15 Pivot / unpivot

### Part D: Query Optimization
- [ ] M6-16 Rewriting slow queries
- [ ] M6-17 Index-friendly queries (SARGable)
- [ ] M6-18 EXPLAIN ANALYZE in depth
- [ ] M6-19 Pagination at scale
- [ ] M6-20 Common interview problems (LeetCode-style SQL)
