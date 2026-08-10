---
tags: [ai-agent, agent-skills, prompt-engineering, claude-code, cursor, open-source, multi-agent]
created: 2026-08-10
source: "https://github.com/msitarzewski/agency-agents"
---

# Agency Agents（The Agency）

[msitarzewski/agency-agents](https://github.com/msitarzewski/agency-agents) 维护的开源「AI 顾问团队」花名册：230+ 个带个性、流程与可交付物的专业 Agent Markdown，可一键安装到 Claude Code、Cursor、Codex、OpenCode、Gemini CLI、Hermes 等主流编码 Agent。另有桌面端 [Agency Agents App](https://agencyagents.app)（macOS / Linux / Windows）浏览花名册并自动同步安装。MIT 许可。

## 核心概念

- **不是通用「Act as X」提示词**：每个 Agent 是领域专家包——身份与语气、核心使命、关键规则、技术交付物、工作流、成功指标
- **按事业部（Division）组织**：Engineering、Design、Paid Media、Sales、Marketing、Product、PM、Testing、Security、Support、Spatial Computing、Specialized、Finance、Game Dev、Academic、GIS、Healthcare 等
- **多工具同一源**：源文件为 Claude Code 友好的 `.md`；`convert.sh` / `install.sh` 生成 Cursor `.mdc`、Codex TOML、Antigravity/Osaurus `SKILL.md`、OpenClaw `SOUL.md` 等
- **可组合编排**：多 Agent 并行协作（如 Nexus Spatial Discovery）；`Agents Orchestrator` 负责跨 Agent 协调
- **社区本地化**：简中、日、韩、葡、俄、印尼、阿、越等独立翻译/本地化仓库；中文版含中国市场专属 Agent（小红书、微信、B 站等）

## 与 Skill / Harness 的关系

| 概念 | Agency Agents | 相近项目 |
|------|---------------|----------|
| 形态 | 整份人格化 Agent 提示（身份 + 流程 + 交付物） | [[agent-skills]] 的渐进式 Skill 包（description 触发 + 按需加载） |
| 安装目标 | 多平台 Agent / rules / skills 目录 | [[trellis]] 偏仓库内 spec/task/workflow 脚手架 |
| 运行时 | 依赖宿主（Claude Code / Cursor / …）加载 | [[hermes-agent]]、[[opencode]]、[[grok-build]] 等是完整 Agent runtime |

二者可叠加：Agency 提供「角色剧本」，Skill/Harness 提供触发、上下文注入与任务生命周期。

## 快速上手

### 桌面 App（推荐）

```bash
# macOS
brew install --cask msitarzewski/agency-agents/agency-agents
```

或从 [agency-agents-app releases](https://github.com/msitarzewski/agency-agents-app/releases/latest) 下载；支持 Claude Code、Cursor、Codex、Gemini、OpenCode、Qwen、Osaurus 等。

### 脚本安装

```bash
git clone https://github.com/msitarzewski/agency-agents.git
cd agency-agents

./scripts/convert.sh          # 生成各工具集成文件
./scripts/install.sh          # 交互式：检测已装工具并勾选

# 定向安装
./scripts/install.sh --tool cursor
./scripts/install.sh --tool claude-code --division engineering,security
./scripts/install.sh --tool cursor --agent frontend-developer,ui-designer
./scripts/install.sh --list teams
```

OpenCode 运行时目前约只注册 ~119 个 Agent（上游限制），建议用 `--division` 子集安装。

## 支持的工具与落盘路径

| 工具 | 产物形态 | 典型路径 |
|------|----------|----------|
| Claude Code | 原生 `.md` | `~/.claude/agents/` |
| GitHub Copilot | 原生 `.md` | `~/.github/agents/`、`~/.copilot/agents/` |
| Cursor | `.mdc` rules | 项目 `.cursor/rules/` |
| OpenCode | `.md` agents | 项目 `.opencode/agents/` |
| Codex | TOML custom agents | `~/.codex/agents/` |
| Gemini CLI | `.md` agents | `~/.gemini/agents/` |
| Antigravity / Osaurus | `SKILL.md` | `~/.gemini/config/skills/`、`~/.osaurus/skills/` |
| Aider / Windsurf | 合并单文件 | `CONVENTIONS.md` / `.windsurfrules` |
| OpenClaw | `SOUL.md` + `AGENTS.md` + `IDENTITY.md` | `~/.openclaw/agency-agents/` |
| Hermes | lazy-router plugin | `~/.hermes/plugins/` |
| Qwen / Kimi | SubAgent / YAML | `.qwen/agents/`、`~/.config/kimi/agents/` |

Cursor 用法示例：安装后用 `@security-engineer` 等 rule 引用；Claude Code / Copilot 用自然语言「activate Frontend Developer」。

## Agent 文件结构（贡献模板）

新增 Agent 时按固定骨架：

1. Frontmatter：`name`、`description`、`color`
2. Identity & Memory
3. Core Mission
4. Critical Rules（领域硬约束）
5. Technical Deliverables（含示例）
6. Workflow Process
7. Success Metrics

设计原则：强个性、明确交付物、可度量成功标准、可复用工作流、可积累记忆模式——区别于一次性 prompt 库或黑盒工具。

## 事业部一览（节选）

- **Engineering**：Frontend / Backend / SRE / DevOps / RAG / Identity / i18n / Multi-Agent Systems Architect 等数十个工程角色
- **Design**：UI/UX、Brand Guardian、Whimsy Injector、UI Finish-Gate Reviewer
- **Marketing / Paid Media / Sales**：增长、社媒（含中国平台）、PPC、Outbound、MEDDPICC 等
- **Testing / Security**：Evidence Collector、Reality Checker、AppSec、Pentest、AI-Generated Code Auditor
- **Game Dev**：跨引擎设计 + Unity / Unreal / Godot / Blender / Roblox
- **GIS / Healthcare / Academic / Finance**：垂直领域专家包

完整花名册与用例见源 README；示例见仓库 `examples/`（如八事业部并行的 Nexus Spatial Discovery）。

## 相关链接

- [[agent-skills]] — Skill 规范、渐进式加载与设计模式
- [[trellis]] — 仓库内 spec/task/workflow Agent harness
- [[hermes-agent]] — 可安装 Agency 的 Hermes 运行时
- [[opencode]] — OpenCode 集成目标之一
- [[matt-pocock-skills]] — 另一类可组合工程 Skills
- [[impeccable]] — 设计向 Skill 包（可与 Design Division 对照）
- [[llm-knowledge-bases]] — 把可复用知识编译为持久 Markdown 的范式
