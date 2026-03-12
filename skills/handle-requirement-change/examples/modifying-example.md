# Example: MODIFYING — Changing API Response Format

## Scenario

An e-commerce API returns product lists as flat arrays. The PM requests: "Change the product list endpoint to return paginated responses with metadata."

**Current response:**
```json
[
  {"id": 1, "name": "Widget", "price": 9.99},
  {"id": 2, "name": "Gadget", "price": 19.99}
]
```

**Desired response:**
```json
{
  "data": [...],
  "pagination": {"page": 1, "pageSize": 20, "total": 156, "totalPages": 8}
}
```

---

## Step 1: CLASSIFY

**Change Type:** MODIFYING
**User's words:** "Change the product list endpoint to return paginated responses with metadata."
**Business context:** Product catalog has grown to 10k+ items. Frontend needs pagination.
**Affected area:** Product list endpoint, all consumers of that endpoint.
**Risk:** HIGH — response format change breaks every existing consumer.

---

## Step 2: SURVEY

**Investigation:**

```bash
# Find all consumers of the product list endpoint
Grep: "/api/products" in **/*.{js,ts,tsx}
# Result:
#   src/routes/products.js (the endpoint)
#   src/services/productService.js (business logic)
#   frontend/src/hooks/useProducts.ts (frontend consumer)
#   frontend/src/components/ProductGrid.tsx (renders products)
#   tests/products.test.js (API tests)
#   tests/integration/product-flow.test.js (E2E test)

# Find what the response is used for
Grep: "products\." in frontend/src/**/*.{ts,tsx}
# Result: products.map(), products.length, products.filter() — all assume array
```

**Impact Report Summary:**
- Files directly affected: `src/routes/products.js`, `src/services/productService.js`
- Files transitively affected: `frontend/src/hooks/useProducts.ts`, `frontend/src/components/ProductGrid.tsx`
- Tests to update: `tests/products.test.js` (12 tests), `tests/integration/product-flow.test.js` (3 tests)
- Conflicts: Frontend code assumes array response — MUST update frontend simultaneously
- Existing patterns: Other endpoints (orders, users) also return flat arrays — consider if they need pagination too (flag to user, but out of scope)

**Current behaviors to catalog:**
1. Returns ALL products (no limit) — CHANGE to paginated
2. Supports `?category=X` filter — KEEP
3. Supports `?sort=price` — KEEP
4. Frontend calls `.map()` directly on response — MUST CHANGE
5. Frontend uses `.length` for count display — MUST CHANGE to use `pagination.total`
6. E2E test checks array length — MUST UPDATE

---

## Step 3: SCOPE

**Tasks:**
1. Add pagination helper to `src/utils/pagination.js` — S
2. Update `src/services/productService.js` — add count query, accept page/pageSize params — M
3. Update `src/routes/products.js` — wrap response in paginated format — S
4. Update `frontend/src/hooks/useProducts.ts` — unwrap `data` from response, expose pagination — M
5. Update `frontend/src/components/ProductGrid.tsx` — use `pagination.total` for count, add pagination controls — M
6. Update `tests/products.test.js` — all 12 tests need new response format — M
7. Update `tests/integration/product-flow.test.js` — 3 tests need update — S

**Non-Goals:**
- NOT paginating other endpoints (orders, users)
- NOT changing product detail endpoint format
- NOT adding caching layer

**DO NOT TOUCH:**
- `?category=X` filter logic (just pass through to paginated query)
- `?sort=price` logic (just pass through)
- Product detail endpoint (`/api/products/:id`)
- Order endpoints

**Behaviors to preserve:**
1. Category filtering works identically
2. Sort works identically
3. All existing product data fields present in response
4. Product detail endpoint unchanged

---

## Step 4: IMPLEMENT

Key decisions during implementation:
- Task 2 first: backend pagination logic (testable independently)
- Task 3: route wrapping (backend tests can verify)
- Tasks 4-5: frontend updates (only after backend verified)
- Backend tests updated WITH backend changes (tasks 2-3 include test updates)
- Frontend tests updated WITH frontend changes

At each checkpoint:
- Category filter still works? YES
- Sort still works? YES
- Product fields unchanged? YES
- Detail endpoint untouched? YES

---

## Step 5: VERIFY

- [x] 12 updated API tests pass with new response format
- [x] 3 updated E2E tests pass
- [x] Category filter: `?category=electronics&page=1` returns correct filtered paginated results
- [x] Sort: `?sort=price&page=2` returns correct sorted paginated results
- [x] Product data fields identical to before (just wrapped in `data` array)
- [x] Product detail endpoint returns same format as before (not paginated)
- [x] Frontend renders correctly with pagination controls
- [x] `pagination.total` matches actual product count

---

## Step 6: LEARN

- **Surprise:** Frontend used `.length` for count display — not in any docs.
- **Failed attempt:** Tried updating backend without frontend simultaneously, broke the app.
- **Would-do-differently:** Always check ALL consumers before planning.
