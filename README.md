# AnnoyingPM

A Claude Code skill for systematically handling requirement changes across software development, AI/ML, LLM, RL, and agent development.

## Problem

AI coding agents handle requirement changes poorly — whether it's traditional software, LLM prompts, RL reward functions, or ML pipelines:
- Skip impact analysis and jump straight to implementation
- Break existing features/baselines when making changes
- Don't distinguish between adding, modifying, and rewriting
- Forget to update tests/evals alongside code changes
- Miss implicit behaviors when replacing old code
- **AI/ML specific:** Don't establish baselines before changes, ignore non-deterministic risks, create train-serve skew

## Solution

The `handle-requirement-change` skill enforces a 5-phase process:

1. **CLASSIFY** — What type of change? (additive / modifying / replacing)
2. **SURVEY** — What exists? What will be affected?
3. **SCOPE** — What changes, what doesn't, what must be preserved?
4. **IMPLEMENT** — Incremental coding with checkpoints
5. **VERIFY** — Full validation before claiming done

Each phase has a hard gate — you can't skip ahead.

## Installation

### Per-project (recommended)

Copy the skill into your project:

```bash
cp -r skills/handle-requirement-change/ your-project/.claude/skills/handle-requirement-change/
```

### Global (all projects)

```bash
cp -r skills/handle-requirement-change/ ~/.claude/skills/handle-requirement-change/
```

## Usage

The skill activates automatically when Claude Code detects requirement changes:

**Software:** "Add X feature" / "Change how Y works" / "Rewrite Z"
**LLM:** "Change the system prompt" / "Switch to Claude" / "Add a new tool"
**RL:** "Modify the reward function" / "Add training constraints"
**ML:** "Change the loss function" / "Add new features" / "Switch frameworks"
**Agent:** "Add a new capability" / "Change orchestration logic"

Or invoke it explicitly with `/handle-requirement-change`.

## Structure

```
skills/handle-requirement-change/
├── SKILL.md           # Core skill definition
├── templates/         # Output format templates for each phase
│   ├── change-request.md
│   ├── impact-report.md
│   └── scope-document.md
└── examples/          # Complete walkthroughs
    ├── additive-example.md     # Software: adding auth
    ├── modifying-example.md    # Software: changing API format
    ├── replacing-example.md    # Software: SQL to ORM migration
    └── ai-ml-examples.md      # AI/ML: LLM prompt change, RL reward, new baseline, feature engineering
```

## Examples

### Software Development
- **Additive:** Adding user auth to a todo app
- **Modifying:** Changing API response format to paginated
- **Replacing:** Rewriting data layer from raw SQL to ORM

### AI/ML Development
- **LLM Prompt Change:** Modifying RAG system prompt + output format
- **RL Reward Change:** Switching from sparse to dense shaped reward
- **New Baseline:** Implementing SAC alongside existing PPO
- **Feature Engineering:** Adding features + changing time windows in ML pipeline

## Research & Design

See `docs/design.md` for the research and reasoning behind this skill's design.

## License

MIT
