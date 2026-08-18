---
tags: [lean4, theorem-proving, agent-skills, mathlib, formal-methods, mcp, cursor, claude-code]
created: 2026-08-18
source: "raw/2026-08-18-lean4-skills.md"
---

# Lean 4 Skills

[cameronfreer/lean4-skills](https://github.com/cameronfreer/lean4-skills) 是一套面向 AI 编码 Agent 的 **Lean 4 定理证明工作流包**：把 informal claim → 形式化骨架 → 填 sorry → 审查 / 重构 / golf → checkpoint 编成可复用循环，并附带 mathlib 搜索、公理检查与安全护栏。工作流本身 host-agnostic（Claude Code、Codex、Cursor、Gemini、OpenCode 等共用同一套核心 skill），只有调用表面不同。MIT 许可；作者 Cameron Freer。截至 2026-08，GitHub ~377 stars。

## 核心概念

- **先搜后证**：大量数学事实已在 mathlib 里；每个 sorry 先做 mathlib 搜索，再写 tactic
- **类型检查即测试套件**：`lake build` 通过、约定范围内零 sorry、只用标准公理（`propext` / `Classical.choice` / `Quot.sound`），证明才算完成
- **声明头是契约**：`prove` / `autoprove` 不得改 theorem/lemma 语句与类型签名；改陈述属于 `formalize` / `autoformalize` 合成工作流
- **共享证明循环**：`prove` / `autoprove` / `formalize` / `autoformalize` 共用 **Plan → Work → Checkpoint → Review → Replan → Continue/Stop**
- **三档安装**：Tier 1 只有 SKILL.md + references；Tier 2 可移植 checkout + `lean4-skills-*` helper；Tier 3 原生插件（命令 / hooks / subagents）
- **LSP-first**：优先用 [lean-lsp-mcp](https://github.com/oOo0oOo/lean-lsp-mcp) 做 live goal 与搜索；脚本与全量 `lake build` 是降级/门禁，不是第一反应

## 工作流目录

| 工作流 | 作用 |
|--------|------|
| `draft` | 从非正式断言起草 Lean 声明骨架 |
| `formalize` | 交互式形式化：起草 + 引导证明 |
| `autoformalize` | 从非正式来源端到端自治形式化 |
| `prove` | 按周期引导填 sorry（每轮询问） |
| `autoprove` | 带明确停止预算的多周期自治证明 |
| `disprove` | 引导反例搜索，Lean 类型检查否定后才报 `REFUTED` |
| `checkpoint` | 保存点：单文件 + 项目构建、公理扫描、commit |
| `review` | 只读质量审查（`--mode=batch` 或 `stuck`） |
| `refactor` | 复用 mathlib、抽出 helper、简化证明策略 |
| `golf` | 提升证明的直接性、清晰度、性能与简洁 |
| `learn` | 交互教学与 mathlib 探索（`--mode=repo` / `mathlib`） |
| `diagnose` | 环境、插件、项目诊断与迁移 |

Claude Code 用 `/lean4:<name>`；其他宿主加载 `lean4` skill 后点名工作流即可。七个参数密集命令（`draft` / `learn` / `formalize` / `autoformalize` / `prove` / `autoprove` / `disprove`）由 host-agnostic parser 校验启动参数。

典型会话：`draft`（或 `formalize` / `autoformalize`）→ `prove`（或 `autoprove`）→ `review` → `refactor` → `golf` → `checkpoint` → `git push`。要反驳而不是证明时走 `disprove`。

无命令编辑 `.lean` 时只跑 **一轮有界 pass**（读 goal / 搜 mathlib / 试自动化 tactic / 校验），卡住则进入 Blocked-Goal Triage，再交接给 `prove` / `autoprove`。

## 共享证明循环与质量门

证明引擎共用 cycle engine：每个 sorry 做 mathlib 搜索、tactic 尝试与校验；卡住则强制 review + replan。`prove` 的 replan 需用户批准；`autoprove` 按 `--review-every` 自动继续，直到停止预算。

**质量门（证明完成条件）：**

- `lake build` 通过
- 约定范围内零 sorry
- 只用标准公理
- 未经允许不改陈述

**校验阶梯：** `lean_diagnostic_messages(file)`（每次编辑）→ `lake env lean` 文件门（从项目根）→ 仅在 checkpoint / 终局才 `lake build`。

**自动化 tactic 顺序（成功即停）：** `rfl` → `simp` → `ring` → `linarith` → `nlinarith` → `omega` → `exact?` → `apply?` → `grind` → `aesop`。

**Docstring 规则：** 改已有声明只动证明体注释（Rule A）；`review` 只读提议（Rule B）；`draft` / `formalize` / `autoformalize` 可为新建声明写 docstring（Rule C）。

`disprove` 是另一条分支：6 阶段循环（Plan → Work → Checkpoint → Review → Accumulate → Continue/Stop），只追加 `T_counterexample`，不改原声明；仅当 Lean 类型检查否定时才报 `REFUTED`。

## 安装档位

| 档位 | 得到什么 | 适用 |
|------|----------|------|
| **Tier 1** | `SKILL.md` + references + UI metadata；无 helper / hooks / 命令 | 宿主自带 skill 安装器；Windows 无 POSIX shell 时的保底 |
| **Tier 2** | 一次 clone + `~/.agents/skills/lean4` 链接 + `LEAN4_*` 环境块 + `lean4-skills-*` wrappers | Cursor / Windsurf / OpenCode / Copilot / Gemini 等推荐完整运行时 |
| **Tier 3** | 原生插件：Claude Code 的 `/lean4:*` + subagents；Codex 的 `$lean4` + 受信 hooks + 绝对路径 helper | Claude Code、Codex |

**Claude Code：**

```text
/plugin marketplace add cameronfreer/lean4-skills
/plugin install lean4
```

**Codex：**

```bash
codex plugin marketplace add cameronfreer/lean4-skills --ref main
codex plugin add lean4@lean4-skills
```

Codex 不会自动信任 hooks：先在 `/hooks` 审查并信任 hash，再开新任务让 SessionStart 注入绝对路径。没有 `/lean4:*` 对等命令，用 `$lean4` 点名工作流。

**Cursor（本库相关）：** 从 `.agents/skills/`、`.cursor/skills/`、`~/.agents/skills/`、`~/.cursor/skills/` 发现 skill。推荐 Tier 2 portable checkout；Tier 1 则把 `plugins/lean4/skills/lean4/` 拷到 `.cursor/skills/lean4/`。聊天里输入 `/lean4`，或让 Agent 在 Lean 任务上自动激活。旧的 `.cursor/rules/lean4.mdc` 已过时。Windows 无 symlink 时用 copy 变体，更新后需重新覆盖拷贝。

**v3 → v4（仅 Claude Code）：** 三个插件（`lean4-theorem-proving` / `lean4-memories` / `lean4-subagents`）合并为一个 `lean4`；命令前缀改为 `/lean4:*`；memory 集成已移除。

## Lean LSP MCP 与运行时画像

可选但强烈推荐 [lean-lsp-mcp](https://github.com/oOo0oOo/lean-lsp-mcp)：live goal、mathlib 搜索、通常比反复全量构建快得多。常用工具：`lean_goal`、`lean_local_search`、`lean_leanfinder`、`lean_leansearch`、`lean_loogle`、`lean_hammer_premise`、`lean_multi_attempt`、`lean_diagnostic_messages`。

Skill 按能力降级：

- **full**：MCP + subagents + 命令
- **mcp_main_only**：MCP 只在主线程，不派发 subagent
- **scripts_only**：无 live goal / 无 tactic 试跑 / 无行级诊断，靠 `lean4-skills-smart-search` + `lake env lean`
- **review_only**：只读

Helper 脚本（sorry 分析、公理扫描、mathlib 搜索、golf 检测等）通过 `lean4-skills-*` wrappers 调用；报告类调用加 `--report-only`，checkpoint 等门禁命令不要加。

## 与同类 Skill 包的关系

- **[[agent-skills]]** — 规范层（渐进式加载、description 触发、`references/` 下沉）；本仓库是按该标准分发的领域工作流包
- **[[matt-pocock-skills]]** — 工程纪律（grilling / TDD / deep modules）；本仓库是形式化证明纪律
- **[[impeccable]]** — 设计质量注入；本仓库是 Lean/mathlib 质量注入
- **[[opencode]]** / **[[pi]]** — 可发现 `.agents/skills` 的宿主；本仓库是装进去的领域 skill

## 相关链接

- [[agent-skills]] — Skill 规范结构、渐进式加载与设计模式
- [[matt-pocock-skills]] — 另一套可组合工程 Skill
- [[impeccable]] — 设计向 Skill 包
- [[opencode]] — 发现 `.agents/skills` 的开源编码 Agent
- [[pi]] — 可安装第三方 skill 的极简 harness
- [[codegraph]] — 另一类给 Agent 的代码理解增强（符号图 vs 证明状态）

## 参考

- 仓库: https://github.com/cameronfreer/lean4-skills
- 安装指南: https://github.com/cameronfreer/lean4-skills/blob/main/INSTALLATION.md
- 核心 skill: https://github.com/cameronfreer/lean4-skills/blob/main/plugins/lean4/skills/lean4/SKILL.md
- Lean LSP MCP: https://github.com/oOo0oOo/lean-lsp-mcp
