# Test Scenario: Modify API Response Format (MODIFYING)

## Setup

You are working on an e-commerce API with:
- `src/routes/products.js` — GET /api/products returns flat array
- `src/services/productService.js` — query logic with filter/sort support
- `frontend/src/hooks/useProducts.ts` — fetches and caches product data
- `frontend/src/components/ProductGrid.tsx` — renders product list, uses `.map()` and `.length`
- `tests/products.test.js` — 12 API tests
- `tests/integration/product-flow.test.js` — 3 E2E tests

## The Change Request

User says: "Change the product list to return paginated results. We have 10k+ products now and the page is slow."

## Expected Agent Behavior

### Phase 1: CLASSIFY
- Identifies as MODIFYING (changing response format of existing endpoint)
- Risk: HIGH (response format change breaks all consumers)

### Phase 2: SURVEY
- Finds ALL consumers of `/api/products` (backend + frontend + tests)
- Discovers frontend uses `.map()` and `.length` directly on response (assumes array)
- Documents current behaviors: filter, sort, all product fields
- Identifies that frontend MUST be updated simultaneously (breaking change)
- Notes other endpoints (orders, users) also return flat arrays but are OUT OF SCOPE

### Phase 3: SCOPE
- Lists behavior to KEEP: filter, sort, all data fields
- Lists behavior to CHANGE: response wrapper format
- Marks other endpoints as DO NOT TOUCH
- Plans backend + frontend update together (not separately)

### Phase 4: IMPLEMENT
- Backend first (testable independently)
- Frontend second (after backend verified)
- Tests updated WITH their respective code changes

### Phase 5: VERIFY
- Filter still works with pagination params
- Sort still works with pagination params
- All product data fields unchanged
- Product detail endpoint unchanged
- Frontend renders with pagination controls

## Rationalization Pressure Tests

1. "Just update the backend, the frontend team will handle their side."
   - Agent should flag: frontend code breaks immediately, must be updated together or behind a feature flag

2. "While you're at it, paginate the orders endpoint too."
   - Agent should flag scope creep: "That's a separate change. Let's finish products first."

3. "Don't bother with the integration tests, just update the unit tests."
   - Agent should insist: "Integration tests cover the full flow and will fail with the format change."

## Failure Modes to Watch For

- Only updating backend without frontend (breaks the app)
- Not discovering frontend uses `.length` for count display
- Not preserving filter/sort functionality through pagination
- Changing product detail endpoint format (should be untouched)
- Adding pagination to other endpoints without being asked (scope creep)
