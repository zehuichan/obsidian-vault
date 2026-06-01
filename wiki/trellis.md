---
tags: [ai-agent, spec-driven, agent-skills, workflow, cursor, claude-code, open-source, team-collaboration]
created: 2026-05-26
source: "https://docs.trytrellis.app/"
---

# Trellis

由 Mindfold HQ 维护的**团队级 AI 编码 Agent 脚手架（Agent Harness）+ 仓库内 LLM wiki**。把项目规范（spec）、任务（task）、工作流（workflow）和会话记忆（workspace journal）全部固化为 git 跟踪的 Markdown，并通过 hook、auto-trigger skill、sub-agent 三类原语自动注入到任意 AI 编码会话。当前覆盖 14 个主流编码 Agent（Claude Code、Cursor、Codex、OpenCode、Gemini CLI、Copilot、Windsurf、Kiro、Kilo、Antigravity、Qoder、CodeBuddy、Droid、[[pi|Pi Agent]]），并通过 `.agents/skills/` 标准兼容更多 Agent。安装：`npm install -g @mindfoldhq/trellis@beta`。

> 比喻：AI 的能力像藤蔓，生命力旺盛但四处蔓延；Trellis 就是引导藤蔓沿你团队规范生长的支架。

## 核心概念

- **Spec**：项目编码规范，按主题模块化拆分到 `.trellis/spec/`，按任务精准注入（而不是一整个 `CLAUDE.md` / `.cursorrules` 全文塞进去）
- **Task**：单次工作单元，含 PRD、研究文档、子任务 JSONL、状态/分支/PR 元数据，落在 `.trellis/tasks/<id>/`
- **Workspace**：每位开发者的 session journal，写在 `.trellis/workspace/<name>/journal-N.md`，让 AI 跨会话记住"上次干到哪"
- **Workflow**：`.trellis/workflow.md` 是单一可执行契约，包含三阶段定义、skill 路由表、按 task 状态注入的 `<workflow-state>` 提示、`task.py` 16 个子命令清单
- **三类原语**：Command（用户触发的会话边界）/ Skill（平台按意图自动匹配的阶段工作流）/ Sub-agent（主会话生成的隔离子角色）

## 架构：Agent Harness + 内置 LLM wiki

### Agent Harness 层

- **三阶段工作流**：Plan / Execute / Finish，在 `workflow.md` 中显式定义
- **Per-turn breadcrumb**：hook 脚本 `inject-workflow-state.py` 在每次 `UserPromptSubmit` 解析当前任务 `status`，把对应 `<workflow-state>` 块注入到本轮提示
- **任务生命周期**：需求、状态、负责人、分支、PR 元数据、子任务关系全部存盘
- **三种角色分离**：`trellis-research`（调查）/ `trellis-implement`（写代码）/ `trellis-check`（验证）
- **Read-before-write**：implement/check 路径强制先读 PRD 与相关 spec 才允许改文件
- **Finish 边界**：最终验证、spec 更新、提交、归档、写 journal 五个步骤分离，避免"边写边收尾"

### 内置 LLM wiki 层

| 内容 | 位置 |
|------|------|
| 项目规范与思维指南 | `.trellis/spec/` |
| 任务 PRD / 研究 / JSONL 清单 | `.trellis/tasks/` |
| 个人 journal | `.trellis/workspace/<name>/` |
| 可执行工作流契约 | `.trellis/workflow.md` |
| 本地定制说明 | `trellis-meta` references |

这与 [[llm-knowledge-bases]] 提出的"把可复用知识编译进仓库 Markdown，跨会话持久化"是同一范式。

## 三类原语对照

| 原语 | 谁触发 | 典型用途 |
|------|--------|----------|
| **Command** `/trellis:*` | 用户 | 会话边界：`start`、`finish-work`、`continue` |
| **Skill** | 平台按意图自动匹配 | 阶段工作流：`brainstorm`、`before-dev`、`check`、`update-spec`、`break-loop` |
| **Sub-agent** | 主会话 spawn | 隔离子角色：`implement`、`check`、`research` |

**Skill-first 设计**：原本作为 slash 命令的 `brainstorm` / `before-dev` / `check` / `update-spec` / `break-loop`，0.6 之后全部迁移为 auto-trigger skill，由平台按用户意图匹配，无需手动 `/调用`。这种取舍与 [[agent-skills]] 文章总结的"description 决定触发，正文承载知识"一致。

## 不同平台的兼容性策略

| 平台 | SessionStart hook | Sub-agent | Trellis 适配方式 |
|------|-------------------|-----------|-------------------|
| Claude Code、OpenCode、Gemini CLI、Qoder、CodeBuddy、Copilot、Droid、Pi Agent | 有 | 有 | hook 自动注入 + 真子 Agent |
| Cursor | 无（用 `alwaysApply` rule 替代）| 无 | `.cursor/rules/trellis.mdc` 永远生效；implement/check 在主会话内联运行；skill 由 Cursor 规则系统按内容匹配 |
| Codex | 可选（`codex_hooks = true`）| 有 | 同 Claude Code 路径 |
| Kiro、Kilo、Antigravity、Windsurf | 无 | 部分无 | 需手动 `/trellis:start` 拉取 prelude；sub-agent 工作内联跑 |

这种"按平台能力降级"的写法承认了一个现实：**只有少数 Agent 同时具备 hook 和 sub-agent**，框架必须为每一档都备一条退路。

## 与同类方案的关系

| 维度 | [[openspec]] | Trellis |
|------|--------------|---------|
| 关注层 | 需求 / 功能规格 / 变更审查 | 完整开发循环（规范 + 任务 + 工作流 + 记忆） |
| 核心产物 | spec.md + change proposal + spec delta | spec/ + tasks/ + workspace/ + workflow.md |
| 工作流强制 | 文档约定 | hook + skill + sub-agent 三层强制 |
| 跨会话记忆 | 提案 + spec 留仓 | 加上 per-developer journal |
| 安装方式 | `npm i -g @fission-ai/openspec` | `npm i -g @mindfoldhq/trellis@beta` |

可以把 OpenSpec 看作 Trellis 中 spec/task 子集的轻量先行者；Trellis 则在其上加了**工作流状态机、角色分工、journal、平台适配层**，目标是替代项目里所有的 `.cursorrules` / `CLAUDE.md` / 散落的 skill 集合。

与 [[agent-skills]] 的关系：Trellis 是 skill-first 框架的一个具体实例，验证了 description-driven 自动触发 + 渐进加载在多平台脚手架上的可行性；同时通过 `.agents/skills/` 标准让 skill 资产可在 Trellis 之外被其他 Agent 复用。

与 [[impeccable]] / [[open-design]] 的关系：后两者是垂直领域（编码命令规范、UI 设计）的 skill 包，Trellis 则提供承载这类 skill 的通用 harness，三者在 Cursor / Claude Code 上可并存。

## 设计要点

- **单一可执行契约**：把 phase 定义、skill 路由表、per-turn 提示、命令清单全部塞进 `workflow.md` 一个文件；fork = 改一个 markdown，无需改 Python 或重发版
- **Hook 只做解析**：`inject-workflow-state.py` 不内嵌任何 fallback 文案，只读取 `workflow.md` 中的 `[workflow-state:STATUS]` 块，避免"两处真实来源"
- **任务即上下文配置**：`task.py create` 同时写 `status=planning` 并设置当前任务指针，确保 `[workflow-state:planning]` 从下一轮就开始注入，brainstorm 期间也有约束
- **Read-before-write 是硬性约束**：不是建议，是 implement/check 路径里的强制步骤，正面应对"AI 跳过文档直接改代码"的常见失败模式

## 风险与局限

- **学习成本**：14 个平台 + 3 类原语 + 多种 hook/prelude 模式，文档体量大；快速试用与深度定制的台阶比 OpenSpec 高
- **平台能力差异**：Cursor 没有 sub-agent，"implement / check 内联运行"实际意味着 Cursor 上的 Trellis 体验不如 Claude Code 完整
- **Beta 期**：0.6 还在 beta；workflow.md 的契约形态、`task.py` 子命令集仍在演进
- **过度仪式化风险**：三阶段 + 多 skill + journal 写入，对小型独立项目可能比直接用 [[agent-skills]] 或 [[openspec]] 更重

## 相关链接

- 官网 / 文档：[docs.trytrellis.app](https://docs.trytrellis.app/)
- 文档索引（给 LLM 用）：[llms.txt](https://docs.trytrellis.app/llms.txt)
- 安装：`npm install -g @mindfoldhq/trellis@beta`
- 同范式概念：[[openspec]]、[[llm-knowledge-bases]]、[[agent-skills]]
- 同生态 Skill 包：[[impeccable]]、[[open-design]]
- 底层基建参考：[[codegraph]]（结构化代码索引，可与 Trellis 的 spec/task 体系互补）
