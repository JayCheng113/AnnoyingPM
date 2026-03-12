# Example: REPLACING — Rewriting Data Access Layer from Raw SQL to ORM

## Scenario

A Node.js app uses raw SQL queries throughout route handlers. The PM requests: "Rewrite the data layer to use Prisma ORM. We're tired of SQL injection risks and manual migrations."

---

## Phase 1: CLASSIFY

**Change Type:** REPLACING
**User's words:** "Rewrite the data layer to use Prisma ORM. We're tired of SQL injection risks and manual migrations."
**Business context:** Security concern (SQL injection), developer productivity (manual migrations).
**Affected area:** ALL database interactions across the entire application.
**Risk:** HIGHEST — touching every data access path in the app.

---

## Phase 2: SURVEY

**Investigation:**

```bash
# Find all raw SQL usage
Grep: "db\.query\|db\.run\|db\.all\|db\.get" in **/*.{js,ts}
# Result: 47 matches across 12 files

# Find all database models/tables referenced
Grep: "FROM\|INTO\|UPDATE\|DELETE FROM" in **/*.{js,ts}
# Result: Tables: users, todos, categories, sessions, audit_log

# Find all migration files
Glob: **/migrations/**
# Result: 8 migration files

# Find all tests
Glob: **/*.test.{js,ts}
# Result: 24 test files, 180 test cases

# Check for transactions
Grep: "BEGIN\|COMMIT\|ROLLBACK\|transaction" in **/*.{js,ts}
# Result: 3 transaction blocks in src/services/orderService.js
```

**Complete Old Behavior Inventory:**

### Explicit Behaviors
1. CRUD operations for 5 tables (users, todos, categories, sessions, audit_log)
2. JOIN queries: todos-with-categories, users-with-sessions
3. Transactions in order processing (3 operations atomic)
4. Soft deletes on users (sets `deleted_at`, doesn't actually delete)
5. Audit logging on every write operation
6. Full-text search on todos (`LIKE '%term%'`)

### Implicit Behaviors (CRITICAL — not in any spec)
1. `audit_log` INSERT happens in same transaction as data changes
2. Session cleanup cron job uses raw `DELETE WHERE expires_at < NOW()`
3. Category counts are computed via `COUNT(*)` subquery, not cached
4. User lookup by email is case-insensitive (`LOWER(email)`)
5. Todo ordering defaults to `created_at DESC` even when not specified
6. Soft-deleted users are excluded from ALL queries via `WHERE deleted_at IS NULL`

**Conflicts:** None with other requirements, but Prisma soft-delete needs explicit middleware configuration.

---

## Phase 3: SCOPE

**Tasks (ordered by dependency):**
1. Initialize Prisma, create schema from existing tables — M
2. Write Prisma migration from existing SQL migrations — M
3. Create data access layer: `src/db/repositories/` for each table — L
4. Replace `src/services/userService.js` SQL with Prisma repository — M
5. Replace `src/services/todoService.js` SQL with Prisma repository — M
6. Replace `src/services/categoryService.js` SQL with Prisma repository — S
7. Replace `src/services/sessionService.js` SQL with Prisma repository — S
8. Replace `src/services/orderService.js` SQL with Prisma (preserve transactions!) — M
9. Replace `src/services/auditService.js` SQL with Prisma — S
10. Update all test files to use Prisma test utilities — L
11. Remove old SQL migration files after verifying Prisma migrations work — S
12. Update session cleanup cron to use Prisma — S

**DO NOT TOUCH:**
- Route handlers (they call services, services call DB — only services change)
- API response formats
- Authentication middleware logic
- Business logic in services (only data access changes)

**Behaviors to preserve (ALL from inventory above):**
1. All CRUD operations identical output
2. Transactions atomic in order processing
3. Soft deletes on users with global filter
4. Audit logging in same transaction as data changes
5. Case-insensitive email lookup
6. Default todo ordering `created_at DESC`
7. Full-text search functionality
8. Session cleanup behavior

**Parallel implementation strategy:**
- Keep old SQL code running until new Prisma code is verified per-service
- Don't delete old code in same task as writing new code

---

## Phase 4: IMPLEMENT

**Key rule: old code stays until new code is verified.**

For each service (tasks 4-9):
1. Write Prisma repository alongside existing SQL code
2. Write/update tests against Prisma repository
3. Tests pass? Switch service to use Prisma repository
4. Verify all service tests still pass
5. Only after ALL services migrated: remove old SQL utilities

**Checkpoint after each service migration:**
- [ ] Same data returned for same inputs?
- [ ] Transactions preserved?
- [ ] Soft delete filter applied?
- [ ] Audit logging works in transaction?
- [ ] No route handler changes needed?

---

## Phase 5: VERIFY

- [x] All 180 test cases pass (updated for Prisma)
- [x] Transaction atomicity: order processing rolls back correctly on failure
- [x] Soft delete: `GET /users` excludes soft-deleted, `GET /admin/users?includeDeleted=true` includes them
- [x] Audit log entries created in same transaction as data changes
- [x] Case-insensitive email: `user@EXAMPLE.com` finds `user@example.com`
- [x] Default ordering: todos without `?sort=` param return `created_at DESC`
- [x] Full-text search: `?q=meeting` finds todos containing "meeting"
- [x] Session cleanup cron runs correctly with Prisma
- [x] No route handler was modified (diff confirms)
- [x] No API response format changed (integration tests confirm)
- [x] Old SQL files removed
- [x] Prisma migrations generate identical schema to old SQL migrations
