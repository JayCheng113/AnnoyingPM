# AnnoyingPM

[English](README.md)

> 你的 PM 又改需求了？让 AI agent 也能优雅地应对。

一个遵循 [Agent Skills](https://agentskills.io) 开放标准的技能，让 AI 编码 agent 系统化地处理需求变更——覆盖传统软件开发、LLM 应用、强化学习、ML 训练管线和 AI agent 开发。兼容 **Claude Code**、**Codex CLI** 及所有支持 Agent Skills 标准的工具。

## 为什么需要这个

AI coding agent 处理需求变更时的典型翻车：

- 跳过影响分析，直接开写 → 改到一半发现漏了关键依赖
- 改了 A 功能，B 功能悄悄挂了 → 没人发现直到用户报 bug
- "就改个 prompt" → 下游解析全崩了，因为输出格式变了
- 改了 reward function → proxy reward 上涨，真实目标原地踏步（reward hacking）
- 换了框架 → "默认参数不一样"的隐形 bug 折腾三天

**根本原因：** agent 把每个需求都当全新任务处理，不看已有代码就开干。

## 怎么解决

一个 6 步流程，按变更大小自动缩放：

```
SIZE → CLASSIFY → SURVEY → SCOPE → IMPLEMENT → VERIFY → LEARN
  ↑                                     |
  └─── mid-stream pivot? re-scope ──────┘
```

| 步骤 | 做什么 | 门控 |
|------|--------|------|
| **SIZE** | 评估复杂度：trivial / small / medium / large | 决定流程深度 |
| **CLASSIFY** | 分类：新增 / 修改 / 替换 / 优化 + 为什么 | 类型和原因清楚 |
| **SURVEY** | 找出所有受影响的文件、测试、冲突 | 影响范围明确 |
| **SCOPE** | 任务列表 + 不动区域 + Non-Goals + 保留行为 | 边界画清 |
| **IMPLEMENT** | 增量实现，每步验证 | 每个任务通过 |
| **VERIFY** | 全量测试 + 范围覆盖检查 | 全绿 |
| **LEARN** | 记录意外、失败尝试、改进建议 | 知识沉淀 |

**关键特性：**

- **按大小缩放** — 改个 typo 直接做，改个框架走完整流程
- **强制自检** — 实现前 5 个必答问题，替代被动的"注意事项"列表（激活率从 20% 提升到 84%+）
- **Non-Goals** — 明确"这次不解决什么"，从意图层面防止 scope creep
- **Mid-stream Pivots** — 开发到一半改需求？三种路径：微调 / 换方向 / 新需求
- **Failed Attempts 记录** — Sionic AI 的研究表明：失败记录比成功路径被引用更多
- **领域专项指导** — LLM / RL / ML / Agent / 框架迁移，每个领域有具体命令和常见踩坑表

## 安装

本技能遵循 [Agent Skills](https://agentskills.io) 开放标准，支持多种工具。

### Claude Code

```bash
# 项目级别（推荐）
cp -r skills/handle-requirement-change/ your-project/.claude/skills/handle-requirement-change/

# 全局（所有项目生效）
cp -r skills/handle-requirement-change/ ~/.claude/skills/handle-requirement-change/
```

### Codex CLI

```bash
# 项目级别（推荐）
cp -r skills/handle-requirement-change/ your-project/.agents/skills/handle-requirement-change/

# 全局（所有项目生效）
cp -r skills/handle-requirement-change/ ~/.agents/skills/handle-requirement-change/
```

### 同一项目同时支持两个工具

```bash
# 安装到 Claude Code
cp -r skills/handle-requirement-change/ your-project/.claude/skills/handle-requirement-change/

# 为 Codex CLI 创建符号链接（避免重复）
mkdir -p your-project/.agents/skills
ln -s ../../.claude/skills/handle-requirement-change your-project/.agents/skills/handle-requirement-change
```

### 兼容性

本技能使用标准 SKILL.md 格式（含 YAML frontmatter）。工具特有的 frontmatter 字段会被不支持的工具自动忽略。

| 工具 | 技能路径 | 触发方式 |
|------|---------|---------|
| Claude Code | `.claude/skills/` | `/handle-requirement-change` 或自动检测 |
| Codex CLI | `.agents/skills/` | 需求变更时自动检测 |
| VS Code Copilot | `.agents/skills/` | 自动检测 |
| Cursor | `.cursor/skills/` 或 `.agents/skills/` | 自动检测 |

## 使用

Skill 会在检测到需求变更时自动激活：

```
"加个 X 功能"          → ADDITIVE
"改一下 Y 的行为"       → MODIFYING
"用 B 重写 A"          → REPLACING
"优化 Z 的性能"         → OPTIMIZING
"等等，改成用 Redis"     → Mid-stream pivot
```

也可以手动调用：`/handle-requirement-change`

### 适用场景

| 领域 | 示例 |
|------|------|
| 软件开发 | 加功能、改接口、重构、换技术栈 |
| LLM / Prompt | 改 system prompt、换模型、加 tool、改输出格式 |
| RL 训练 | 改 reward function、换 curriculum、调环境参数 |
| ML 管线 | 改 loss、加数据源、换特征工程、改模型架构 |
| AI Agent | 加工具、改编排逻辑、调 context 管理、加安全护栏 |

## 项目结构

```
skills/handle-requirement-change/
├── SKILL.md                  # 核心流程（171行，agent 直接读这个）
├── domain-guidance.md        # 领域专项指导（按需加载，不浪费 context）
├── templates/
│   └── change-checklist.md   # 轻量级 checklist（仅 LARGE 变更使用）
└── examples/                 # 完整场景演示
    ├── additive-example.md   # 给 todo app 加认证
    ├── modifying-example.md  # 改 API 响应格式为分页
    ├── replacing-example.md  # SQL 迁移到 ORM
    └── ai-ml-examples.md    # 4个AI/ML场景
```

## 设计原则

**1. 按大小缩放**

不是所有变更都需要完整流程。改个 typo 就直接改，改个框架才需要正式文档。

**2. 从失败中学习**

每次 MEDIUM+ 变更后记录：什么出乎意料、什么尝试失败了、下次怎么改进。灵感来自 [Sionic AI](https://huggingface.co/blog/sionic-ai/claude-code-skills-training) 的 `/retrospective` 模式——他们用这个方法每天跑 1000+ ML 实验，知识不断积累。

**3. Non-Goals 防止蔓延**

"DO NOT TOUCH" 告诉你别改哪些文件。Non-Goals 告诉你这次变更不解决什么问题。后者在意图层面更有效——来自 [AGENTS.md 标准](https://github.com/agentsmd/agents.md)对 2500+ 仓库的分析。

## 研究来源

这个 skill 的设计综合了多个社区最佳实践：

| 来源 | 借鉴了什么 |
|------|-----------|
| [Superpowers](https://github.com/obra/superpowers) (53k+ stars) | Iron Law、Hard Gate、Red Flags 模式 |
| [Sionic AI](https://huggingface.co/blog/sionic-ai/claude-code-skills-training) | Failed Attempts 表、LEARN 步骤、/retrospective 模式 |
| [AGENTS.md](https://github.com/agentsmd/agents.md) | Non-Goals 模式 |
| [Forced-eval 研究](https://scottspence.com/posts/how-to-make-claude-code-skills-activate-reliably) | Pre-Implementation Self-Check（激活率 84%+） |
| [Addy Osmani](https://addyosmani.com/blog/good-spec/) | Spec 作为活文档、mid-stream pivot 处理 |
| [awesome-cursorrules](https://github.com/PatrickJS/awesome-cursorrules) | 三层边界模型、领域特化规则 |

详细设计文档见 `docs/design.md`。

## 版本演进

| 版本 | 改动 |
|------|------|
| v1 | 初始版本：5阶段硬门控流程 + 三种变更类型 |
| v2 | 实用化：复杂度分级、SKILL.md 468→174行、Mid-stream Pivots |
| v3 | 社区最佳实践：强制自检、Non-Goals、LEARN步骤、Failed Attempts |
| v4 | 场景审查：OPTIMIZING类型、无测试处理、LARGE分解、pivot细分 |
| v5 | 跨工具兼容：Agent Skills 标准、支持 Claude Code + Codex CLI + Cursor |

## License

MIT
