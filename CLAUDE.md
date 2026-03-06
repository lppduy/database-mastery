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

## Progress
- [x] M1-01 Full Table Scan
- [x] M1-02 Write Amplification (index + high write load)
- [ ] M1-03 Composite Index Wrong Order
- [ ] M1-04 Write Amplification Cost
