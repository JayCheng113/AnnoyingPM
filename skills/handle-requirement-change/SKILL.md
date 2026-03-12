---
name: handle-requirement-change
description: Use when requirements change mid-stream, new features are requested, existing behavior needs modification, or a component needs rewriting - across software, AI/ML, LLM, RL, or agent development
---

# Handle Requirement Change

Analyze before implementing. Verify before claiming done. Learn from every change.

## The Iron Law

```
NO IMPLEMENTATION WITHOUT IMPACT ANALYSIS.
NO CLAIMING DONE WITHOUT VERIFYING PRESERVED BEHAVIORS.
```

## Step 0: SIZE

| Size | Criteria | Process |
|------|----------|---------|
| **TRIVIAL** | Single file, no behavior change (typo, comment) | Just do it. |
| **SMALL** | 1-3 files, isolated, no dependents | Mental walkthrough of steps 1-3, implement, quick verify |
| **MEDIUM** | Multiple files, shared code, has dependents | All steps with inline notes |
| **LARGE** | Cross-module, architectural, migration | All steps with formal checklist (`templates/change-checklist.md`) |

**Default UP, not down.** If unsure between SMALL and MEDIUM → choose MEDIUM.

**AI/ML rule:** Non-deterministic components (prompts, rewards, model configs) = automatically MEDIUM+.

## Step 1: CLASSIFY

| Type | Signal | Key Risk |
|------|--------|----------|
| **ADDITIVE** | "Add X" — nothing equivalent exists | Integration conflicts |
| **MODIFYING** | "Change how X works" — behavior changes | Breaking dependents |
| **REPLACING** | "Rewrite X" / "Switch A to B" | Losing implicit behaviors |

Capture: **WHY** this change is being made. Business context drives scope decisions.

## Step 2: SURVEY

<HARD-GATE>
No implementation until survey is done.
</HARD-GATE>

**ALL changes (SMALL+):**
1. Grep/Glob for ALL files referencing affected code — do NOT rely on memory
2. Identify tests covering the affected area
3. Check for conflicts with existing requirements

**Add for MODIFYING/REPLACING:**
4. Read and summarize current behavior BEFORE touching anything
5. Map every caller/consumer of the code being changed

**Add for REPLACING:**
6. Inventory old code's behaviors — explicit AND implicit (undocumented edge cases)

**AI/ML (pick relevant):**
- LLM: Map prompt chain, check output format contracts, document eval baselines
- RL: Record training metrics, checkpoint current policy, check env-agent interface
- ML: Map data flow end-to-end, check train-serve consistency, document model baselines
- Migration: Map ALL API surface (not just main functions), check implicit framework defaults

**Gate:** You can list affected files, tests, and conflicts. For MODIFYING/REPLACING: you can describe current behavior.

## Step 3: SCOPE

1. **Task list** — specific changes, in dependency order
2. **DO NOT TOUCH** — files/modules that stay unchanged
3. **Non-Goals** — what this change explicitly does NOT address
   (e.g., "NOT a performance optimization" / "NOT changing the data model" / "NOT upgrading the framework")
4. **Preserve list** (MODIFYING/REPLACING) — behaviors that MUST still work
5. **Scope creep check** — exceeds original request? Flag it.

**Gate:** Tasks, boundaries, non-goals, and preservation requirements are clear.

## Pre-Implementation Self-Check

BEFORE writing any code, answer these explicitly. If ANY answer is NO → STOP.

1. Can I list ALL affected files? → if no: Step 2 incomplete
2. Can I describe current behavior of code I'm changing? → if no: Step 2 incomplete
3. Do I have a DO NOT TOUCH list? → if no: Step 3 incomplete
4. Do I have Non-Goals defined? → if no: Step 3 incomplete
5. Am I about to change something "because I'm already here"? → if yes: scope creep, re-scope

## Step 4: IMPLEMENT

1. Follow task order from Step 3
2. After each task: run relevant tests (existing + new)
3. Update tests and docs WITH the code, not after
4. MODIFYING/REPLACING: new behavior verified BEFORE old behavior deleted
5. Scope changing? STOP → return to Step 3

**Checkpoint per task:**
- [ ] Follows existing codebase patterns?
- [ ] Tests pass (including existing)?
- [ ] DO NOT TOUCH respected?
- [ ] (AI/ML) Metrics compared against baselines?

## Step 5: VERIFY

<HARD-GATE>
Do NOT claim done until verified.
</HARD-GATE>

1. Full test suite passes (not just changed tests)
2. All Step 3 scope items addressed
3. All DO NOT TOUCH items actually untouched
4. All Non-Goals confirmed not accidentally addressed (scope stayed focused)
5. MODIFYING/REPLACING: each preserved behavior verified
6. Docs/specs updated

## Step 6: LEARN (MEDIUM+ changes)

After completing, capture briefly:

1. **What surprised you** — unexpected dependencies, implicit behaviors discovered
2. **Failed attempts** — what you tried that didn't work and WHY
   *(Most valuable part — future agents reference failures more than successes)*
3. **Would-do-differently** — process improvements for next time

LARGE: write as a note with the commit. MEDIUM: flag surprises to the user verbally.

## Mid-Stream Pivots

When requirements change DURING implementation:

1. **STOP** — do not adapt on the fly
2. **Classify the pivot:** new change (finish current first) or modification to current change (re-survey)
3. **Check conflicts** between what you've built and the new direction
4. **Re-scope** (Step 3) — do NOT just tack onto existing scope
5. **Resume** from re-scoped task list

**The trap:** "Almost done, let me squeeze this in." → This is how regressions happen.

## Domain-Specific Guidance

Read on demand — only the section relevant to your current change:
- `domain-guidance.md` — LLM/Prompt, RL/Training, ML Pipeline, AI Agent, Framework Migration
  Each domain includes: concrete commands, common failed attempts, and verification patterns.

## Templates

- `templates/change-checklist.md` — unified lightweight checklist for LARGE changes
- SMALL/MEDIUM: mental walkthrough or inline notes. No formal documents needed.

## Quick Reference

| Step | Action | Gate |
|------|--------|------|
| 0. SIZE | Trivial/Small/Medium/Large | Determines process depth |
| 1. CLASSIFY | Additive/Modifying/Replacing + why | Type + reason clear |
| 2. SURVEY | Files, tests, conflicts, behaviors | Impact understood |
| 3. SCOPE | Tasks + DO NOT TOUCH + Non-Goals + preservations | Boundaries drawn |
| Self-Check | 5 questions before coding | All YES |
| 4. IMPLEMENT | Incremental with checkpoints | Each task verified |
| 5. VERIFY | Full test suite + scope coverage | All green |
| 6. LEARN | Surprises + failed attempts + improvements | Knowledge captured |
