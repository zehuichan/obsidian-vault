---
tags: [llm, agent, spec-driven, planning, open-source, cursor, brownfield]
created: 2026-05-20
source: "https://openspec.dev/"
---

# OpenSpec

由 Fission AI 维护的**轻量级 spec-driven 开发框架**：把功能需求以 Markdown 规格写在仓库里，每次变更生成 **spec delta**、提案、设计与任务分解，让审查者先看「意图变化」再看代码。开源、无需 API Key、不依赖 MCP；通过 `npm install -g @fission-ai/openspec@latest` 安装，对 Cursor、Claude Code、Codex、Copilot 等 20+ 编码 Agent 提供原生 slash 命令集成。

## 核心概念

- **Spec 即活文档**：按能力（capability）组织在 `openspec/specs/<capability>/spec.md`，与代码同仓、可 git 协作
- **Spec delta**：变更以 diff 形式展示需求增删（Requirement + GIVEN/WHEN/THEN Scenario），审查「意图」而非逐行读实现
- **Change 包**：`/openspec:proposal` 生成 `openspec/changes/<id>/`，含 `proposal.md`、`design.md`、`tasks.md` 及 `specs/` 下的 delta
- **Brownfield-first**：面向已有代码库，按需增量建 spec，反对一次性全量生成
- **Agent 无关**：规格不绑定某一 LLM 产品，换 Agent 仍可沿用同一套规划层

## 目录结构

```
openspec/specs/           # 按能力划分的现行规格
├── auth-session/spec.md
└── checkout-payment/spec.md

openspec/changes/<id>/    # 单次变更的审查包
├── proposal.md
├── design.md
├── tasks.md
└── specs/                # 对该变更影响的 spec delta
```

规格正文常用 **SHALL** 表述需求，用 **Scenario**（GIVEN / WHEN / THEN）描述可验收行为，类似轻量 BDD。

## 工作流示例

用户输入类似：

```
/openspec:proposal Add remember me checkbox with 30-day sessions
```

Agent 会：检索现有 `auth-session` spec → 扫描 `src/auth/session.ts` → 产出 `add-remember-me` 变更目录，并汇总「影响 1 个 spec、3 个阶段、8 个任务」。团队在写代码前审阅 proposal / design / tasks 与 spec delta，提前对齐。

## 与 Plan Mode、其他规划工具的区别

| 维度 | Agent 内置 Plan Mode | OpenSpec |
|------|----------------------|----------|
| 生命周期 | 单次对话 | 跨会话、可 PR 共享 |
| 产出物 | 聊天内计划 | 仓库内 proposal + spec delta |
| 需求留存 | 通常丢弃 | 合并进 `openspec/specs/` 活文档 |
| 场景假设 | 偏绿地 | 偏棕地（弄清现有系统） |

官方强调：**轻流程**（几分钟想清楚再开写）、**非瀑布**（计划足够好即可，变更时更新 spec）。

## 路线图：Workspaces

团队向功能开发中，针对大型单体/多仓、定制集成与协作增强；面向工程团队早期试用。

## 与本知识库的关系

OpenSpec 的「规格进仓库、Agent 读 spec 执行」与 [[llm-knowledge-bases]] 的「wiki 进仓库、Agent 维护知识」同属 **文件驱动、跨会话持久上下文** 范式；与 [[agent-skills]] 互补——Skill 教 Agent *怎么做*，OpenSpec 记录系统 *应该做什么*。在 Cursor 上可与 [[impeccable]]、[[open-design]] 等 Agent 增强工具并存：前者偏 UI/设计质量，OpenSpec 偏需求与变更审查。

## 相关链接

- 官网：[openspec.dev](https://openspec.dev/)
- GitHub：[Fission-AI/OpenSpec](https://github.com/Fission-AI/OpenSpec/)
- Discord：[社区](https://discord.gg/YctCnvvshC)
- 相关概念：[[llm-knowledge-bases]]、[[agent-skills]]
- 同生态：[[open-design]]、[[impeccable]] — 均强调 Agent 可移植与仓库内 Markdown 资产，但 OpenSpec 专注**功能规格与变更审查**而非设计 Skill
