---
tags: [ai-agent, agent-skills, coding-agent, tdd, domain-driven-design, open-source, workflow]
created: 2026-07-23
source: "raw/2026-07-23-matt-pocock-skills.md"
---

# Matt Pocock Skills

Matt Pocock 开源的 **Agent Skills 工具包**（口号「Skills For Real Engineers」）：把软件工程基本功——对齐需求、共享领域语言、反馈环、持续设计——压缩成可组合、可改写的小 skill，而不是像 GSD / BMAD / Spec-Kit 那样接管整条流程。MIT 许可；可通过 [skills.sh](https://skills.sh/mattpocock/skills) 拷贝进项目自改，或作为 Claude Code 插件订阅更新。截至 2026-07，GitHub ~183k stars，最新版 v1.1.0。

## 核心概念

- **小而可组合**：刻意不做「拥有全过程」的重型框架；skill 可单独选用、改写、与任意模型配合
- **User-invoked vs Model-invoked**：用户 slash 命令负责编排（如 `/grill-me`）；模型可自动调用的 skill 承载可复用纪律（如 `/tdd`）。用户 skill 可以调用模型 skill，但不会互相调用
- **Grilling 对齐**：用穷尽式访谈缩小人机意图落差，再动手改代码
- **共享语言（CONTEXT.md + ADR）**：项目术语表降低废话、统一命名、减少思考 token
- **反馈环优先**：静态类型、浏览器、自动化测试；尤其是 red-green-refactor 的 `/tdd`
- **日常设计投资**：用 deep modules 思维对抗「Agent 加速熵增」；`/improve-codebase-architecture` 定期扫泥球

## 要解决的四类失败模式

| # | 症状 | 核心修复 |
|---|------|----------|
| 1 | Agent 没做你想要的 | `/grill-me` / `/grill-with-docs` 先对齐 |
| 2 | Agent 太啰嗦 | `CONTEXT.md` 共享领域语言 + ADR |
| 3 | 代码跑不通 | `/tdd` + `/diagnosing-bugs` + 类型/浏览器反馈 |
| 4 | 变成泥球架构 | `/to-spec` 问清模块边界 + `/improve-codebase-architecture` |

其中 `#2` 被作者称为可能是仓库里「最酷」的技巧：把「某课程某节里的 lesson 被 materialize 到文件系统」压缩成「materialization cascade」这类术语后，跨会话都更短、更准。

## 安装

**skills.sh（可编辑副本）**：

```bash
npx skills@latest add mattpocock/skills
```

选 skill 与目标 Agent 时务必包含 `/setup-matt-pocock-skills`，然后在仓库里跑一次：配置 issue tracker（GitHub / Linear / 本地文件）、triage 标签、文档落盘位置。

**Claude Code 插件（只读、随上游更新）**：

```
/plugin marketplace add mattpocock/skills
/plugin install mattpocock-skills@mattpocock
```

两种哲学：skills.sh = fork 后自己改；plugin = 订阅作者维护的包。Codex 等符合 Agent Skills 标准的 harness 已可走 skills.sh；原生 Codex 插件在路线图上。

## Skill 目录速查

### Engineering — User-invoked

| Skill | 作用 |
|-------|------|
| `ask-matt` | 路由器：该用哪条 flow |
| `grill-with-docs` | 访谈 + 领域模型，更新 CONTEXT.md / ADR |
| `triage` | 按角色状态机推进 issue |
| `improve-codebase-architecture` | 扫描 deepening 机会 → HTML 报告 → 挑一项 grill |
| `setup-matt-pocock-skills` | 每仓库首次配置 |
| `to-spec` | 把当前对话合成 spec，发到 tracker（不访谈） |
| `to-tickets` | 拆成带阻塞边的 tracer-bullet tickets |
| `implement` | 按 spec/tickets 实现，在约定 seam 走 `/tdd`，收尾 `/code-review` |
| `wayfinder` | 超大工作拆成调查票地图，逐张澄清路径 |

### Engineering — Model-invoked

| Skill | 作用 |
|-------|------|
| `prototype` | 可扔掉原型（终端态机 或 多 UI 变体） |
| `diagnosing-bugs` | 复现 → 最小化 → 假设 → 埋点 → 修 → 回归 |
| `research` | 高信任一手源调研，落成带引用的 Markdown（后台 agent） |
| `tdd` | red-green-refactor，按垂直切片推进 |
| `domain-modeling` | 术语表挑战 + 边界场景，更新 CONTEXT/ADR |
| `codebase-design` | deep modules：小接口后大量行为、干净 seam、可测 |
| `code-review` | Standards + Spec 双轴并行子 agent 审查 |
| `resolving-merge-conflicts` | 按意图逐 hunk 解决冲突，不 `--abort` |

### Productivity

- **User-invoked**：`grill-me`、`handoff`、`teach`、`writing-great-skills`
- **Model-invoked**：`grilling`（`grill-me` / `grill-with-docs` 背后的可复用循环）

## 与同类工具的关系

- **[[openspec]] / [[trellis]]** — 偏仓库内活规格与变更包；Matt 的 skills 偏可组合纪律，且明确反对「框架拥有全过程」
- **[[ponytail]]** — 压缩实现层代码量；Matt 更强调对齐、领域语言、TDD 与架构 deepening
- **[[impeccable]]** — 设计质量注入；本仓库是工程纪律注入
- **[[agent-skills]]** — 规范与模式层；本仓库是按该标准分发的一套实战 skill 集合

## 相关链接

- [[agent-skills]] — Skill 规范结构、渐进式加载与设计模式
- [[openspec]] — 另一路 spec-driven 规划层
- [[trellis]] — 团队级 Agent 脚手架 + 仓库内 wiki
- [[ponytail]] — 最小可用代码规则集
- [[impeccable]] — 设计向 Skill 包
- [[lean4-skills]] — 形式化证明向 Skill 包（sorry 循环 / mathlib / 公理门）
- [[pi]] — 可安装第三方 skill 的极简 harness
