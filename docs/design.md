# Design Document: handle-requirement-change Skill

## Problem Statement

AI coding agents struggle with requirement changes because they treat every request as a greenfield task. When a user says "add X" or "change Y," agents typically:

1. Start writing code immediately without analyzing existing code
2. Don't distinguish between adding new code vs modifying existing code vs rewriting
3. Break existing features by not understanding what must be preserved
4. Skip test and documentation updates
5. Don't detect conflicts between new and existing requirements

This leads to frequent rework, regression bugs, and user frustration.

## Research Sources

### Spec-Driven Development
- Addy Osmani's "How to Write a Good Spec for AI Agents" — specifications as single source of truth
- JetBrains Junie blog — spec-driven approach for AI coding
- GitHub's spec-driven development toolkit — open source tools for spec management

### Superpowers Skills Framework
- obra/superpowers (53k+ GitHub stars) — discipline-enforcing skill patterns
- Iron Laws, Red Flags, Rationalization Tables — proven patterns for agent compliance
- TDD-adapted skill creation process

### Cursor Rules & Agent SOPs
- .cursorrules system — declarative rules for agent behavior
- AWS Strands Agent SOPs — RFC 2119 constraint language (MUST/SHOULD/MAY)
- Continue.dev awesome-rules — community-curated agent rules

### Scope & Conflict Management
- PMI research: 52% of projects experience scope creep
- Multi-agent approaches for requirements elicitation
- Conflict identification and consensus-building prompts

## Key Design Decisions

### Why a single skill instead of three?

Requirement changes are a single workflow with branching paths based on change type. Splitting into separate add/modify/replace skills would:
- Force users to pre-classify before invoking (they often don't know yet)
- Lose the shared classification step that catches misidentification
- Duplicate the survey/scope/verify phases across three files

### Why hard gates instead of suggestions?

Research and testing show that agents routinely skip analysis when given the choice. The superpowers framework demonstrates that hard gates with explicit rationalization rejection are the most effective pattern for enforcement. Suggestions get rationalized away; gates cannot be passed without compliance.

### Why templates as separate files?

Keeps SKILL.md under 500 lines per superpowers guidance. Templates are loaded on demand — not every invocation needs to read the impact report template (e.g., purely additive changes may use a lighter version).

### Why three change types?

Each type has fundamentally different risk profiles and required investigation depth:
- ADDITIVE: main risk is integration point conflicts — need medium survey depth
- MODIFYING: main risk is breaking dependent code — need high survey depth
- REPLACING: main risk is losing implicit behaviors — need highest survey depth

A one-size-fits-all approach either under-investigates replacements or over-investigates additions.

### Why "behaviors to preserve" is mandatory for MODIFYING/REPLACING?

The most common agent failure mode is changing target behavior while unknowingly breaking side-effects. By requiring an explicit preservation checklist, the agent must catalog existing behaviors BEFORE making changes, creating a verification target for Phase 5.

## Architecture

```
Phase 1: CLASSIFY
    ↓ (gate: type + reason + area)
Phase 2: SURVEY
    ↓ (gate: impact report complete)
Phase 3: SCOPE
    ↓ (gate: scoped task list ready)
Phase 4: IMPLEMENT → (scope change?) → back to Phase 3
    ↓ (all tasks done)
Phase 5: VERIFY
    ↓ (gate: all tests pass + all scope covered + preserved behaviors verified)
DONE
```

The bidirectional arrow between IMPLEMENT and SCOPE handles scope discovery during implementation — a common real-world scenario where you find additional work needed only after starting.

## Influence from Existing Skills

- **systematic-debugging**: 4-phase structure with hard gates, rationalization tables
- **brainstorming**: classification step, checklist format, user approval gates
- **test-driven-development**: Iron Law pattern, Red Flags section, "no exceptions" enforcement
- **verification-before-completion**: final verification gate, evidence-before-assertions principle
- **writing-plans**: task decomposition, dependency ordering, file-level specificity
