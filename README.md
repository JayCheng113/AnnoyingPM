# AnnoyingPM

> Your PM changed the requirements again? Let your AI agent handle it gracefully.
>
> 你的 PM 又改需求了？让 AI agent 也能优雅地应对。

A Claude Code skill that helps AI coding agents systematically handle requirement changes — covering traditional software development, LLM apps, reinforcement learning, ML pipelines, and AI agent development.

一个 Claude Code skill，让 AI 编码 agent 系统化地处理需求变更——覆盖传统软件开发、LLM 应用、强化学习、ML 训练管线和 AI agent 开发。

## Why This Exists / 为什么需要这个

Typical failures when AI coding agents handle requirement changes:

AI coding agent 处理需求变更时的典型翻车：

- Skips impact analysis, starts coding → discovers missed dependencies halfway through
  跳过影响分析，直接开写 → 改到一半发现漏了关键依赖
- Changes feature A, feature B silently breaks → nobody notices until users report bugs
  改了 A 功能，B 功能悄悄挂了 → 没人发现直到用户报 bug
- "Just tweaking a prompt" → downstream parsing breaks because output format changed
  "就改个 prompt" → 下游解析全崩了，因为输出格式变了
- Changed reward function → proxy reward goes up, real objective flatlines (reward hacking)
  改了 reward function → proxy reward 上涨，真实目标原地踏步（reward hacking）
- Switched frameworks → hidden bugs from "different default parameters" cost three days
  换了框架 → "默认参数不一样"的隐形 bug 折腾三天

**Root cause:** Agents treat every requirement as a brand-new task without examining existing code.

**根本原因：** agent 把每个需求都当全新任务处理，不看已有代码就开干。

## How It Works / 怎么解决

A 6-step process that auto-scales to change complexity:

一个 6 步流程，按变更大小自动缩放：

```
SIZE → CLASSIFY → SURVEY → SCOPE → IMPLEMENT → VERIFY → LEARN
  ↑                                     |
  └─── mid-stream pivot? re-scope ──────┘
```

| Step / 步骤 | What It Does / 做什么 | Gate / 门控 |
|------|--------|------|
| **SIZE** | Assess complexity: trivial / small / medium / large | Determines process depth |
| | 评估复杂度：trivial / small / medium / large | 决定流程深度 |
| **CLASSIFY** | Categorize: additive / modifying / replacing / optimizing + why | Type and reason clear |
| | 分类：新增 / 修改 / 替换 / 优化 + 为什么 | 类型和原因清楚 |
| **SURVEY** | Identify all affected files, tests, conflicts | Impact scope clear |
| | 找出所有受影响的文件、测试、冲突 | 影响范围明确 |
| **SCOPE** | Task list + do-not-touch zones + Non-Goals + preserved behaviors | Boundaries drawn |
| | 任务列表 + 不动区域 + Non-Goals + 保留行为 | 边界画清 |
| **IMPLEMENT** | Incremental implementation, verify each step | Each task passes |
| | 增量实现，每步验证 | 每个任务通过 |
| **VERIFY** | Full test suite + scope coverage check | All green |
| | 全量测试 + 范围覆盖检查 | 全绿 |
| **LEARN** | Record surprises, failed attempts, improvement suggestions | Knowledge captured |
| | 记录意外、失败尝试、改进建议 | 知识沉淀 |

**Key Features / 关键特性：**

- **Scales to size / 按大小缩放** — Fix a typo? Just do it. Swap a framework? Full process.
  改个 typo 直接做，改个框架走完整流程
- **Forced self-check / 强制自检** — 5 mandatory questions before implementation, replacing passive "watch out for" lists (activation rate from 20% to 84%+)
  实现前 5 个必答问题，替代被动的"注意事项"列表（激活率从 20% 提升到 84%+）
- **Non-Goals** — Explicitly state "what we're NOT solving this time", preventing scope creep at the intent level
  明确"这次不解决什么"，从意图层面防止 scope creep
- **Mid-stream Pivots** — Requirements change mid-development? Three paths: tweak / redirect / new requirement
  开发到一半改需求？三种路径：微调 / 换方向 / 新需求
- **Failed Attempts log** — Sionic AI's research shows failed attempts are referenced more than successful paths
  Sionic AI 的研究表明：失败记录比成功路径被引用更多
- **Domain-specific guidance / 领域专项指导** — LLM / RL / ML / Agent / framework migration, each with concrete commands and common pitfall tables
  每个领域有具体命令和常见踩坑表

## Installation / 安装

### Project-level (recommended) / 项目级别（推荐）

```bash
cp -r skills/handle-requirement-change/ your-project/.claude/skills/handle-requirement-change/
```

### Global (all projects) / 全局（所有项目生效）

```bash
cp -r skills/handle-requirement-change/ ~/.claude/skills/handle-requirement-change/
```

## Usage / 使用

The skill activates automatically when it detects requirement changes:

Skill 会在检测到需求变更时自动激活：

```
"Add feature X"              → ADDITIVE
"Change Y's behavior"        → MODIFYING
"Rewrite A using B"          → REPLACING
"Optimize Z's performance"   → OPTIMIZING
"Wait, switch to Redis"      → Mid-stream pivot
```

You can also invoke it manually / 也可以手动调用：`/handle-requirement-change`

### Applicable Scenarios / 适用场景

| Domain / 领域 | Examples / 示例 |
|------|------|
| Software Dev / 软件开发 | Add features, change APIs, refactor, switch tech stacks / 加功能、改接口、重构、换技术栈 |
| LLM / Prompt | Change system prompts, switch models, add tools, modify output formats / 改 system prompt、换模型、加 tool、改输出格式 |
| RL Training / RL 训练 | Change reward functions, switch curriculum, adjust env params / 改 reward function、换 curriculum、调环境参数 |
| ML Pipeline / ML 管线 | Change loss, add data sources, switch feature engineering, modify architecture / 改 loss、加数据源、换特征工程、改模型架构 |
| AI Agent | Add tools, change orchestration logic, adjust context management, add guardrails / 加工具、改编排逻辑、调 context 管理、加安全护栏 |

## Project Structure / 项目结构

```
skills/handle-requirement-change/
├── SKILL.md                  # Core process (171 lines, agent reads this directly)
│                               核心流程（171行，agent 直接读这个）
├── domain-guidance.md        # Domain-specific guidance (loaded on demand, saves context)
│                               领域专项指导（按需加载，不浪费 context）
├── templates/
│   └── change-checklist.md   # Lightweight checklist (LARGE changes only)
│                               轻量级 checklist（仅 LARGE 变更使用）
└── examples/                 # Full scenario demos / 完整场景演示
    ├── additive-example.md   # Add auth to a todo app / 给 todo app 加认证
    ├── modifying-example.md  # Change API response to paginated / 改 API 响应格式为分页
    ├── replacing-example.md  # Migrate SQL to ORM / SQL 迁移到 ORM
    └── ai-ml-examples.md    # 4 AI/ML scenarios / 4个AI/ML场景
```

## Design Principles / 设计原则

**1. Scale to size / 按大小缩放**

Not all changes need the full process. Fix a typo? Just fix it. Swap a framework? That needs formal documentation.

不是所有变更都需要完整流程。改个 typo 就直接改，改个框架才需要正式文档。

**2. Learn from failures / 从失败中学习**

After every MEDIUM+ change, record: what was surprising, what failed, what to improve next time. Inspired by [Sionic AI](https://huggingface.co/blog/sionic-ai/claude-code-skills-training)'s `/retrospective` pattern — they run 1000+ ML experiments daily with this approach, continuously accumulating knowledge.

每次 MEDIUM+ 变更后记录：什么出乎意料、什么尝试失败了、下次怎么改进。灵感来自 [Sionic AI](https://huggingface.co/blog/sionic-ai/claude-code-skills-training) 的 `/retrospective` 模式——他们用这个方法每天跑 1000+ ML 实验，知识不断积累。

**3. Non-goals prevent creep / Non-Goals 防止蔓延**

"DO NOT TOUCH" tells you which files to leave alone. Non-Goals tells you which problems this change won't solve. The latter is more effective at the intent level — from the [AGENTS.md standard](https://github.com/agentsmd/agents.md)'s analysis of 2500+ repos.

"DO NOT TOUCH" 告诉你别改哪些文件。Non-Goals 告诉你这次变更不解决什么问题。后者在意图层面更有效——来自 [AGENTS.md 标准](https://github.com/agentsmd/agents.md)对 2500+ 仓库的分析。

## Research Sources / 研究来源

The design of this skill synthesizes multiple community best practices:

这个 skill 的设计综合了多个社区最佳实践：

| Source / 来源 | What We Borrowed / 借鉴了什么 |
|------|-----------|
| [Superpowers](https://github.com/obra/superpowers) (53k+ stars) | Iron Law, Hard Gate, Red Flags patterns |
| [Sionic AI](https://huggingface.co/blog/sionic-ai/claude-code-skills-training) | Failed Attempts table, LEARN step, /retrospective pattern |
| [AGENTS.md](https://github.com/agentsmd/agents.md) | Non-Goals pattern |
| [Forced-eval research](https://scottspence.com/posts/how-to-make-claude-code-skills-activate-reliably) | Pre-Implementation Self-Check (84%+ activation rate) |
| [Addy Osmani](https://addyosmani.com/blog/good-spec/) | Spec as living document, mid-stream pivot handling |
| [awesome-cursorrules](https://github.com/PatrickJS/awesome-cursorrules) | Three-layer boundary model, domain-specific rules |

For detailed design documentation, see `docs/design.md`.

详细设计文档见 `docs/design.md`。

## Version History / 版本演进

| Version / 版本 | Changes / 改动 |
|------|------|
| v1 | Initial: 5-stage hard-gated process + 3 change types / 初始版本：5阶段硬门控流程 + 三种变更类型 |
| v2 | Practical: complexity scaling, SKILL.md 468→174 lines, mid-stream pivots / 实用化：复杂度分级、SKILL.md 468→174行、Mid-stream Pivots |
| v3 | Community best practices: forced self-check, Non-Goals, LEARN step, Failed Attempts / 社区最佳实践：强制自检、Non-Goals、LEARN步骤、Failed Attempts |
| v4 | Scenario review: OPTIMIZING type, no-test handling, LARGE decomposition, pivot refinement / 场景审查：OPTIMIZING类型、无测试处理、LARGE分解、pivot细分 |

## License

MIT
