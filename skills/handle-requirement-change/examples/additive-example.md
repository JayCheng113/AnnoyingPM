# Example: ADDITIVE — Adding User Authentication to a Todo App

## Scenario

A todo app has basic CRUD operations. The PM requests: "We need user authentication — users should log in before accessing their todos."

---

## Step 1: CLASSIFY

**Change Type:** ADDITIVE
**User's words:** "We need user authentication — users should log in before accessing their todos."
**Business context:** Multi-user support required. Currently all todos are shared/anonymous.
**Affected area:** Auth system (new), API middleware, todo routes, database schema.
**Risk:** Medium — new code, but touches all existing routes via middleware.

---

## Step 2: SURVEY

**Investigation:**

```bash
# Find all route files
Glob: **/routes/**/*.{js,ts}
# Result: src/routes/todos.js, src/routes/health.js

# Find all middleware
Grep: "middleware" in **/*.{js,ts}
# Result: src/app.js uses cors(), json() middleware

# Find database schema
Glob: **/migrations/**
# Result: migrations/001_create_todos.sql

# Find existing tests
Glob: **/*.test.{js,ts}
# Result: tests/todos.test.js (20 test cases)
```

**Impact Report Summary:**
- Files directly affected: `src/app.js` (add auth middleware), `src/routes/todos.js` (add user scoping)
- Files transitively affected: `tests/todos.test.js` (all tests need auth context)
- Existing patterns: Express + middleware chain, SQL migrations, Jest tests
- Conflicts: None — auth is new, no existing auth code to clash with
- Coverage gaps: No tests for unauthorized access (doesn't exist yet)

**Current behavior documented:** Todos API is open, no auth, all todos visible to all requests.

---

## Step 3: SCOPE

**Tasks (ordered):**
1. Add user migration (`migrations/002_create_users.sql`) — S
2. Add user-todo relation migration (`migrations/003_add_user_to_todos.sql`) — S
3. Create auth module (`src/auth/index.js`) — login, register, JWT verify — M
4. Create auth routes (`src/routes/auth.js`) — POST /login, POST /register — M
5. Add auth middleware to `src/app.js` — S
6. Update `src/routes/todos.js` to scope by user — M
7. Update `tests/todos.test.js` with auth context — M
8. Add `tests/auth.test.js` — M

**Non-Goals:**
- NOT adding role-based permissions (just user-level auth for now)
- NOT migrating existing anonymous todos to users
- NOT implementing OAuth/social login

**DO NOT TOUCH:**
- `src/routes/health.js` — health check should remain unauthenticated
- Todo CRUD logic — only add user scoping, don't change business logic

**Integration points:**
- Auth middleware → existing Express middleware chain in `src/app.js`
- User ID → todo queries in `src/routes/todos.js`

---

## Step 4: IMPLEMENT

Each task executed in order. After each:
- Follows existing patterns (Express router style, migration naming, Jest patterns)
- Tests run and pass
- No changes to "DO NOT TOUCH" items

---

## Step 5: VERIFY

- [x] All 20 original todo tests pass (with auth context added)
- [x] 15 new auth tests pass
- [x] Health endpoint still works without auth
- [x] Todo CRUD logic unchanged (only user scoping added)
- [x] Migrations apply cleanly
- [x] README updated with auth instructions

---

## Step 6: LEARN

- **Surprise:** Existing tests all assumed no auth context — took longer to update than expected.
- **Would-do-differently:** Estimate test update effort in scope.
