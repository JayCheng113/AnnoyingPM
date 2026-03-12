# Test Scenario: Add Authentication to Existing App (ADDITIVE)

## Setup

You are working on a simple Express.js todo app with:
- `src/routes/todos.js` — CRUD endpoints (GET/POST/PUT/DELETE /api/todos)
- `src/routes/health.js` — GET /health (unauthenticated)
- `src/app.js` — Express setup with cors + json middleware
- `tests/todos.test.js` — 20 test cases for todo CRUD
- `migrations/001_create_todos.sql` — todos table

## The Change Request

User says: "Add user login. Each user should only see their own todos."

## Expected Agent Behavior

### Phase 1: CLASSIFY
- Identifies as ADDITIVE (auth doesn't exist yet)
- Notes that existing todo routes will need modification (integration points)
- Risk: Medium

### Phase 2: SURVEY
- Reads `src/routes/todos.js`, `src/app.js`
- Greps for middleware patterns, database patterns
- Identifies that ALL 20 tests assume no auth context
- Identifies `health.js` should remain unauthenticated
- Documents existing patterns (Express router, middleware chain, migration format)

### Phase 3: SCOPE
- Creates ordered task list with auth module first, then middleware, then route updates
- Explicitly marks `health.js` as DO NOT TOUCH
- Marks todo CRUD business logic as DO NOT TOUCH (only add user scoping)

### Phase 4: IMPLEMENT
- Follows existing Express patterns for new routes
- Updates tests alongside code
- Runs tests after each task

### Phase 5: VERIFY
- All original tests pass (with auth context)
- Health endpoint still works without auth
- Todo CRUD unchanged except user scoping

## Rationalization Pressure Tests

Try these prompts to test if the agent resists skipping phases:

1. "Just add a JWT middleware to app.js and update the todo routes. Keep it simple."
   - Agent should still do Phase 2 survey (even if asked to be quick)

2. "We're in a hurry, skip the impact analysis."
   - Agent should push back: "5 minutes of analysis prevents hours of debugging"

3. "I already know what files need to change, just start coding."
   - Agent should verify with Grep/Glob anyway

## Failure Modes to Watch For

- Skipping Phase 2 because "auth is new, nothing to analyze"
- Not identifying that health endpoint should stay unauthenticated
- Not updating existing tests with auth context
- Changing todo CRUD logic instead of just adding user scoping
- Not following existing Express middleware patterns for auth
