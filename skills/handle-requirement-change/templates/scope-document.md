# Scope Document — Phase 3 Output

## Change Reference

- **Change Type:** [ ADDITIVE / MODIFYING / REPLACING ]
- **Affected Area:** [from Phase 1]

## Task List (ordered by dependency)

| # | Task | Files | Estimated Size | Dependencies |
|---|------|-------|---------------|-------------|
| 1 | [description] | `file1.ext`, `file2.ext` | [ S / M / L ] | None |
| 2 | [description] | `file3.ext` | [ S / M / L ] | Task 1 |

## DO NOT TOUCH Boundaries

These files/modules/behaviors are explicitly OUT OF SCOPE. Do not modify them.

| Item | Why It's Out of Scope |
|------|----------------------|
| `path/to/file.ext` | [reason] |
| [behavior X] | [reason — not part of this change request] |

## Behaviors to Preserve (MODIFYING/REPLACING)

These behaviors MUST continue working identically after the change.

| # | Behavior | How to Verify |
|---|----------|--------------|
| 1 | [description] | [test name or manual check] |
| 2 | [description] | [test name or manual check] |

## Behaviors to Change

| # | Current Behavior | New Behavior | Reason |
|---|-----------------|-------------|--------|
| 1 | [what it does now] | [what it should do] | [why] |

## Behaviors to Add (ADDITIVE/MODIFYING)

| # | New Behavior | Integration Point | Test Plan |
|---|-------------|-------------------|----------|
| 1 | [description] | [where it connects to existing code] | [how to test] |

## Scope Creep Check

- [ ] All tasks directly serve the original request
- [ ] No "while we're at it" additions
- [ ] No unrelated refactoring included
- [ ] If scope expanded: user approved the expansion

## Integration Points

| Existing Code | New/Changed Code | Interface |
|--------------|-----------------|----------|
| `existing.ext` | `new.ext` | [how they connect] |

## Rollback Plan

If the change causes unexpected issues:
- [How to revert — e.g., git revert, feature flag, etc.]
