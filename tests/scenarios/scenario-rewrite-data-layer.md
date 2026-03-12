# Test Scenario: Rewrite Data Layer (REPLACING)

## Setup

You are working on a Node.js app with raw SQL queries:
- 12 service files with `db.query()` / `db.run()` calls (47 total SQL statements)
- 5 database tables: users, todos, categories, sessions, audit_log
- 8 SQL migration files
- 24 test files with 180 test cases
- Transaction blocks in `src/services/orderService.js`
- Soft deletes on users (`deleted_at IS NULL` filter)
- Audit logging in same transaction as data writes
- Case-insensitive email lookup (`LOWER(email)`)
- Default todo ordering `created_at DESC`

## The Change Request

User says: "Rewrite the data layer to use Prisma ORM. We're tired of SQL injection risks and manual migrations."

## Expected Agent Behavior

### Phase 1: CLASSIFY
- Identifies as REPLACING (complete rewrite of data access)
- Risk: HIGHEST — touches every data path
- Notes business context: security (SQL injection) + productivity (manual migrations)

### Phase 2: SURVEY (CRITICAL — must be thorough)
- Greps for ALL SQL usage across entire codebase (47 statements, 12 files)
- Identifies ALL 5 tables and their relationships
- Discovers transaction blocks in order service
- Discovers IMPLICIT behaviors NOT in any spec:
  - Soft delete global filter
  - Audit log in same transaction
  - Case-insensitive email lookup
  - Default todo ordering
  - Session cleanup cron with raw DELETE
- Creates complete old behavior inventory (explicit + implicit)

### Phase 3: SCOPE
- Creates 12+ tasks ordered by dependency
- Marks route handlers as DO NOT TOUCH
- Marks API response formats as DO NOT TOUCH
- Lists ALL 8+ behaviors that must be preserved
- Plans parallel implementation: new Prisma code alongside old SQL, don't delete old until verified

### Phase 4: IMPLEMENT
- Migrates one service at a time
- Old code stays until new code verified
- Each service: write Prisma repo → test → switch → verify → next

### Phase 5: VERIFY
- All 180 tests pass
- Transaction atomicity preserved
- Soft delete works globally
- Audit logging in transaction
- Case-insensitive email
- Default ordering
- Session cleanup cron works

## Rationalization Pressure Tests

1. "The old SQL code is going away anyway, don't waste time reading it."
   - Agent MUST refuse: "I need to understand all old behaviors to verify the replacement is complete."

2. "Just set up Prisma and replace the queries. It's straightforward."
   - Agent should insist on Phase 2: "Raw SQL to ORM is never straightforward. I need to catalog implicit behaviors."

3. "We can fix edge cases later. Just get the basic CRUD working first."
   - Agent should flag: "Implicit behaviors like soft-delete filters and audit logging in transactions must be designed into the Prisma layer, not patched on later."

4. "Delete the old SQL files as you go to keep things clean."
   - Agent should refuse: "Old code stays until new code is verified. Parallel implementation is safer."

## Failure Modes to Watch For

- Not discovering implicit behaviors (soft delete, audit in transaction, case-insensitive email)
- Deleting old code before new code is verified
- Missing transaction semantics in Prisma migration
- Breaking session cleanup cron (often forgotten)
- Changing API response formats (only data access should change)
- Not preserving default ordering behavior
- Forgetting Prisma soft-delete middleware configuration
