---
name: handle-requirement-change
description: Use when requirements change mid-stream, new features are requested, existing behavior needs modification, or a component needs rewriting - across software, AI/ML, LLM, RL, or agent development
---

# Handle Requirement Change

Requirement changes break things when agents skip impact analysis and jump straight to implementation.

**Core principle:** Analyze before implementing. Verify before claiming done. Scale the process to the change size.

**Violating the letter of this process is violating the spirit of handling requirement changes.**

## The Iron Law

```
NO IMPLEMENTATION WITHOUT IMPACT ANALYSIS FIRST.
NO CLAIMING DONE WITHOUT VERIFYING PRESERVED BEHAVIORS.
```

## Step 0: SIZE the Change

Before anything else, assess complexity. This determines how much process you need.

| Size | Criteria | Process |
|------|----------|---------|
| **TRIVIAL** | Single file, no behavior change (typo, formatting, comment) | Skip to implementation. No phases needed. |
| **SMALL** | 1-3 files, isolated change, no dependent modules | Mental walkthrough of phases 1-3 (no written artifacts), implement, quick verify |
| **MEDIUM** | Multiple files, touches shared code, has dependents | All 5 phases with inline notes (no formal templates needed) |
| **LARGE** | Cross-module, architectural, framework migration, data pipeline overhaul | All 5 phases with formal output artifacts using templates |

**The trap:** Defaulting everything to SMALL. If you're unsure, size UP not down.

**AI/ML sizing note:** Changes to non-deterministic components (prompts, reward functions, model configs) are automatically at least MEDIUM — non-determinism amplifies hidden impact.

## Step 1: CLASSIFY

What kind of change is this?

| Type | Signal | Key Risk |
|------|--------|----------|
| **ADDITIVE** | "Add X" — nothing equivalent exists | Integration point conflicts |
| **MODIFYING** | "Change how X works" — existing code stays, behavior changes | Breaking dependent modules |
| **REPLACING** | "Rewrite X" / "Switch from A to B" | Losing implicit behaviors |

Also capture: **WHY** is this change being made? (Business context drives scope decisions later.)

## Step 2: SURVEY

<HARD-GATE>
No implementation code until survey is done. Reading code is NOT implementing. Writing/modifying files IS.
</HARD-GATE>

**For ALL changes (SMALL and up):**
1. Grep/Glob for all files referencing the affected code
2. Identify tests covering the affected area
3. Check for conflicts with existing requirements

**Add for MODIFYING/REPLACING:**
4. Read and summarize current behavior BEFORE touching anything
5. Map every caller/consumer of the code being changed

**Add for REPLACING:**
6. Create inventory of old code's behaviors — explicit AND implicit (the undocumented edge cases that will bite you)

**AI/ML additions (pick relevant ones):**
- LLM: Map prompt chain, check output format contracts, document eval baselines
- RL: Record training metrics baseline, check env-agent interface, checkpoint current policy
- ML pipeline: Map full data flow, check train-serve consistency, document model baselines
- Framework migration: Map ALL API surface used (not just main functions), check implicit defaults
- New baseline: Document comparison metrics, verify eval infrastructure compatibility

**Gate:** You can list affected files, tests, and conflicts. For MODIFYING/REPLACING: you can describe current behavior.

## Step 3: SCOPE

Draw boundaries:

1. **Task list** — what specific changes, in what order
2. **DO NOT TOUCH** — explicit list of what stays unchanged
3. **Preserve list** (MODIFYING/REPLACING) — behaviors that MUST still work after the change
4. **Scope creep check** — does this exceed the original request? Flag it.

**Gate:** You have tasks, boundaries, and preservation requirements clear (written or mental depending on SIZE).

## Step 4: IMPLEMENT

**Rules:**
1. Follow task order from Step 3
2. After each task: run relevant tests (existing + new)
3. Update tests and docs WITH the code, not after
4. For MODIFYING/REPLACING: new behavior verified working BEFORE old behavior deleted
5. Scope changing? STOP → return to Step 3

**Checkpoint after each task:**
- [ ] Follows existing codebase patterns?
- [ ] Tests pass (including existing)?
- [ ] DO NOT TOUCH boundaries respected?
- [ ] (AI/ML) Metrics compared against baselines?

## Step 5: VERIFY

<HARD-GATE>
Do NOT claim done until verified.
</HARD-GATE>

1. Run FULL test suite (not just changed tests)
2. All scope items from Step 3 addressed?
3. All DO NOT TOUCH items actually untouched?
4. For MODIFYING/REPLACING: each preserved behavior verified
5. Docs/specs updated

## Mid-Stream Pivots

The most common (and dangerous) scenario: **you're already implementing when requirements change again.**

```
"Actually, let's change that to..."
"Oh wait, we also need..."
"Scratch that part, instead do..."
```

**When this happens:**
1. **STOP implementing immediately** — do not try to adapt on the fly
2. **Assess: is this a new change or a modification to the current change?**
   - New change → finish current work first, then start new change from Step 0
   - Modification to current change → go back to Step 2, re-survey with new requirement
3. **Check for conflicts** between what you've already built and the new direction
4. **Re-scope** (Step 3) with the updated requirements — DO NOT just tack on to existing scope
5. **Resume implementation** from the re-scoped task list

**The trap:** "I'm almost done, let me just squeeze this in." This is how regressions happen. A 2-minute re-scope prevents a 30-minute debugging session.

## Red Flags — STOP and Return to Appropriate Step

| If you're thinking... | The reality is... |
|----------------------|-------------------|
| "Small change, skip analysis" | Small changes in coupled code cause the worst regressions |
| "I already know what's affected" | Use Grep/Glob to verify. Memory misses transitive deps. |
| "I'll update tests after" | "After" never comes. |
| "Old code is being replaced, don't need to read it" | Can't verify replacement without understanding original |
| "It's additive, nothing existing is affected" | New features interact at integration points |
| "User said be quick" | 5 min analysis prevents hours of debugging |
| "I'm mid-implementation, going back is wasteful" | Sunk cost fallacy. Incomplete analysis = more rework. |
| "It's just a prompt/config change" | Non-deterministic components = higher risk, not lower |
| "I'll just run a quick training to check" | Without baseline comparison, you learn nothing |
| "Published results prove it works" | Published ≠ your setup. Reproduce first. |

## Domain-Specific Guidance

For detailed domain-specific guidance, read the relevant section on demand:
- `domain-guidance.md` — LLM/Prompt, RL/Training, ML Pipeline, AI Agent, Framework Migration

Keep SKILL.md focused on the universal process. Domain files add the "what to watch for" per domain.

## Templates

For LARGE changes, use formal output templates:
- `templates/change-request.md` — Step 1 output
- `templates/impact-report.md` — Step 2 output
- `templates/scope-document.md` — Step 3 output

For SMALL/MEDIUM: mental walkthrough or inline notes. Don't fill templates for a 3-file change.

## Quick Reference

| Step | Action | Gate |
|------|--------|------|
| 0. SIZE | Trivial / Small / Medium / Large | Size determines process depth |
| 1. CLASSIFY | Additive / Modifying / Replacing + why | Type + reason clear |
| 2. SURVEY | Find affected files, tests, conflicts | Impact understood |
| 3. SCOPE | Tasks + boundaries + preservations | Boundaries drawn |
| 4. IMPLEMENT | Incremental with checkpoints | Each task verified |
| 5. VERIFY | Full test suite + scope check | All green, all covered |
