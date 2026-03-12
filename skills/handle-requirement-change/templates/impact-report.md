# Impact Report — Phase 2 Output

## Change Reference

- **Change Type:** [ ADDITIVE / MODIFYING / REPLACING ]
- **Affected Area:** [from Phase 1]

## Files Directly Affected

| File | What Changes | Current Purpose |
|------|-------------|----------------|
| `path/to/file.ext` | [description] | [what it does now] |

## Files Transitively Affected

| File | Why Affected | Risk Level |
|------|-------------|-----------|
| `path/to/file.ext` | [imports/calls/depends on affected code] | [ Low / Medium / High ] |

## Test Coverage

| Test File | What It Tests | Status |
|-----------|-------------|--------|
| `path/to/test.ext` | [relevant behavior tested] | [ Covers change / Needs update / Gap ] |

### Coverage Gaps

- [ ] [Behavior that has no test coverage but is affected by this change]

## Documentation Affected

| Document | What Needs Update |
|----------|------------------|
| `path/to/doc.md` | [description] |

## Conflict Analysis

### Conflicts with Existing Requirements

- [ ] **None detected** — New requirement is compatible with all existing behavior
- [ ] **Conflict found:** [description of conflict and which requirements clash]

### Conflicts with In-Progress Work

- [ ] **None detected**
- [ ] **Conflict found:** [description]

## Existing Patterns & Conventions

- **File organization:** [how similar features are organized]
- **Naming conventions:** [patterns observed]
- **Error handling:** [patterns observed]
- **Testing patterns:** [how similar features are tested]

## Current Behavior Summary (MODIFYING/REPLACING only)

[Detailed description of what the existing code does — both explicit features and implicit behaviors. This is the baseline for verifying the change doesn't break anything unintentionally.]

### Explicit Behaviors
1. [behavior]

### Implicit Behaviors (side-effects, edge cases)
1. [behavior]

## Risk Assessment

- **Highest risk area:** [what's most likely to break]
- **Mitigation:** [how to reduce that risk]
