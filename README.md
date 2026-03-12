# AnnoyingPM

[中文版](README_CN.md)

> Your PM changed the requirements again? Let your AI agent handle it gracefully.

A Claude Code skill that helps AI coding agents systematically handle requirement changes — covering traditional software development, LLM apps, reinforcement learning, ML pipelines, and AI agent development.

## Why This Exists

Typical failures when AI coding agents handle requirement changes:

- Skips impact analysis, starts coding → discovers missed dependencies halfway through
- Changes feature A, feature B silently breaks → nobody notices until users report bugs
- "Just tweaking a prompt" → downstream parsing breaks because output format changed
- Changed reward function → proxy reward goes up, real objective flatlines (reward hacking)
- Switched frameworks → hidden bugs from "different default parameters" cost three days

**Root cause:** Agents treat every requirement as a brand-new task without examining existing code.

## How It Works

A 6-step process that auto-scales to change complexity:

```
SIZE → CLASSIFY → SURVEY → SCOPE → IMPLEMENT → VERIFY → LEARN
  ↑                                     |
  └─── mid-stream pivot? re-scope ──────┘
```

| Step | What It Does | Gate |
|------|--------|------|
| **SIZE** | Assess complexity: trivial / small / medium / large | Determines process depth |
| **CLASSIFY** | Categorize: additive / modifying / replacing / optimizing + why | Type and reason clear |
| **SURVEY** | Identify all affected files, tests, conflicts | Impact scope clear |
| **SCOPE** | Task list + do-not-touch zones + Non-Goals + preserved behaviors | Boundaries drawn |
| **IMPLEMENT** | Incremental implementation, verify each step | Each task passes |
| **VERIFY** | Full test suite + scope coverage check | All green |
| **LEARN** | Record surprises, failed attempts, improvement suggestions | Knowledge captured |

**Key Features:**

- **Scales to size** — Fix a typo? Just do it. Swap a framework? Full process.
- **Forced self-check** — 5 mandatory questions before implementation, replacing passive "watch out for" lists (activation rate from 20% to 84%+)
- **Non-Goals** — Explicitly state "what we're NOT solving this time", preventing scope creep at the intent level
- **Mid-stream Pivots** — Requirements change mid-development? Three paths: tweak / redirect / new requirement
- **Failed Attempts log** — Sionic AI's research shows failed attempts are referenced more than successful paths
- **Domain-specific guidance** — LLM / RL / ML / Agent / framework migration, each with concrete commands and common pitfall tables

## Installation

### Project-level (recommended)

```bash
cp -r skills/handle-requirement-change/ your-project/.claude/skills/handle-requirement-change/
```

### Global (all projects)

```bash
cp -r skills/handle-requirement-change/ ~/.claude/skills/handle-requirement-change/
```

## Usage

The skill activates automatically when it detects requirement changes:

```
"Add feature X"              → ADDITIVE
"Change Y's behavior"        → MODIFYING
"Rewrite A using B"          → REPLACING
"Optimize Z's performance"   → OPTIMIZING
"Wait, switch to Redis"      → Mid-stream pivot
```

You can also invoke it manually: `/handle-requirement-change`

### Applicable Scenarios

| Domain | Examples |
|------|------|
| Software Dev | Add features, change APIs, refactor, switch tech stacks |
| LLM / Prompt | Change system prompts, switch models, add tools, modify output formats |
| RL Training | Change reward functions, switch curriculum, adjust env params |
| ML Pipeline | Change loss, add data sources, switch feature engineering, modify architecture |
| AI Agent | Add tools, change orchestration logic, adjust context management, add guardrails |

## Project Structure

```
skills/handle-requirement-change/
├── SKILL.md                  # Core process (171 lines, agent reads this directly)
├── domain-guidance.md        # Domain-specific guidance (loaded on demand, saves context)
├── templates/
│   └── change-checklist.md   # Lightweight checklist (LARGE changes only)
└── examples/                 # Full scenario demos
    ├── additive-example.md   # Add auth to a todo app
    ├── modifying-example.md  # Change API response to paginated
    ├── replacing-example.md  # Migrate SQL to ORM
    └── ai-ml-examples.md    # 4 AI/ML scenarios
```

## Design Principles

**1. Scale to size**

Not all changes need the full process. Fix a typo? Just fix it. Swap a framework? That needs formal documentation.

**2. Learn from failures**

After every MEDIUM+ change, record: what was surprising, what failed, what to improve next time. Inspired by [Sionic AI](https://huggingface.co/blog/sionic-ai/claude-code-skills-training)'s `/retrospective` pattern — they run 1000+ ML experiments daily with this approach, continuously accumulating knowledge.

**3. Non-goals prevent creep**

"DO NOT TOUCH" tells you which files to leave alone. Non-Goals tells you which problems this change won't solve. The latter is more effective at the intent level — from the [AGENTS.md standard](https://github.com/agentsmd/agents.md)'s analysis of 2500+ repos.

## Research Sources

The design of this skill synthesizes multiple community best practices:

| Source | What We Borrowed |
|------|-----------|
| [Superpowers](https://github.com/obra/superpowers) (53k+ stars) | Iron Law, Hard Gate, Red Flags patterns |
| [Sionic AI](https://huggingface.co/blog/sionic-ai/claude-code-skills-training) | Failed Attempts table, LEARN step, /retrospective pattern |
| [AGENTS.md](https://github.com/agentsmd/agents.md) | Non-Goals pattern |
| [Forced-eval research](https://scottspence.com/posts/how-to-make-claude-code-skills-activate-reliably) | Pre-Implementation Self-Check (84%+ activation rate) |
| [Addy Osmani](https://addyosmani.com/blog/good-spec/) | Spec as living document, mid-stream pivot handling |
| [awesome-cursorrules](https://github.com/PatrickJS/awesome-cursorrules) | Three-layer boundary model, domain-specific rules |

For detailed design documentation, see `docs/design.md`.

## Version History

| Version | Changes |
|------|------|
| v1 | Initial: 5-stage hard-gated process + 3 change types |
| v2 | Practical: complexity scaling, SKILL.md 468→174 lines, mid-stream pivots |
| v3 | Community best practices: forced self-check, Non-Goals, LEARN step, Failed Attempts |
| v4 | Scenario review: OPTIMIZING type, no-test handling, LARGE decomposition, pivot refinement |

## License

MIT
