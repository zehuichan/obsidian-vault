---
tags: [ai-agent, coding-agent, terminal, extensible, multi-provider, agent-skills, typescript, open-source]
created: 2026-06-01
source: "https://pi.dev/"
---

# Pi（Pi Coding Agent）

由 Mario Zechner（badlogic）主导、earendil-works 组织开源的**极简、可自我扩展的终端编码 Agent harness**。核心理念是「让工具适应你的工作流，而不是你去适应工具」：内核只保留最小集合（默认 `read`/`write`/`edit`/`bash` 四个工具 + 一个极简 system prompt），刻意**不内置** sub-agent、plan mode、权限弹窗、MCP、待办列表、后台 bash 等「别家都做了的功能」，而是把它们留给 TypeScript 扩展、skill、prompt 模板与 pi package 去实现——甚至可以直接让 pi 改自己的源码、`/reload` 后继续用。纯 TypeScript 单体仓库（pi-mono），MIT 许可；截至 2026-05，GitHub 约 58.3k stars，最新版 v0.78.0（2026-05-29）。

> 官方口号：**Change the harness, not your workflow** · **Primitives, not features**

## 单体仓库：分层的 Agent 工具箱

pi 把「编码 Agent」拆成可独立复用的 npm 包，所以它不只是一个 CLI，更是一套可重组的底层组件：

| 包 | 作用 |
|----|------|
| `@earendil-works/pi-coding-agent` | 交互式编码 Agent CLI（日常用的 `pi` 命令） |
| `@earendil-works/pi-agent-core` | Agent 运行时：工具调用 + 状态管理，可拿来自建 Agent |
| `@earendil-works/pi-ai` | 统一多 provider LLM API，把 OpenAI / Anthropic / Google 等抹平到一个接口 |
| `@earendil-works/pi-tui` | 差分渲染（differential rendering）的终端 UI 库 |
| `@earendil-works/pi-web-ui` | AI 聊天界面的 Web 组件 |

Slack / 聊天自动化与工作流在独立仓库 `earendil-works/pi-chat`。「拆层」的好处：可以用 `pi-agent-core` 在 Node 应用里嵌入 Agent，也能通过 RPC / JSON 事件流被非 Node 系统调用。

## 设计哲学：Primitives, not features（"What we didn't build"）

pi 官网专门有一节列出**故意没做的功能**，并为每一项给出替代路径——这是理解 pi 的关键：

| 没内置 | pi 的态度 / 替代方案 |
|--------|----------------------|
| **MCP** | 用带 README 的 CLI 工具（见 skill），或写扩展加 MCP 支持（作者另有博文论证「你可能不需要 MCP」） |
| **Sub-agents** | 用 tmux 起多个 pi 实例，或用扩展自建，或装现成 package |
| **Plan mode** | 把计划写进文件，或用扩展实现 |
| **权限弹窗** | 跑在容器里，或用扩展按自己的安全需求自建确认流 |
| **内置待办** | 「会让模型犯糊涂」，改用 `TODO.md` 或扩展 |
| **后台 bash** | 用 tmux，可观测、可直接交互 |

这与 [[trellis]]、[[openspec]] 形成有意思的反差：后两者是**往 Agent 里加结构**（工作流状态机 / spec 阶段门），pi 则**把内核压到最小**，让结构成为可选、可替换的扩展。

## 五类自定义原语

pi 的全部扩展能力围绕五种可打包、可分享的原语展开：

| 原语 | 是什么 | 放在哪 |
|------|--------|--------|
| **Extension** | TypeScript 模块，可注册工具 / 命令 / 快捷键 / 事件钩子 / UI 组件，能访问完整 TUI | `~/.pi/agent/extensions/`、`.pi/extensions/` |
| **Skill** | 遵循 [agentskills.io](https://agentskills.io) 标准的按需能力包，`/skill:name` 或由模型自动加载，渐进披露不破坏 prompt cache | `~/.pi/agent/skills/`、`.agents/skills/` 等 |
| **Prompt template** | 可复用提示词 Markdown，输入 `/name` 展开，支持 `{{变量}}` | `~/.pi/agent/prompts/`、`.pi/prompts/` |
| **Theme** | JSON 主题，修改即热重载 | `~/.pi/agent/themes/`、`.pi/themes/` |
| **Pi package** | 把以上四类打包，经 npm 或 git 分享：`pi install npm:@foo/pi-tools` / `pi install git:github.com/badlogic/pi-doom` | `~/.pi/agent/git/`、`~/.pi/agent/npm/` |

> 安全警告（官方原话）：pi package 以**完整系统权限**运行——扩展执行任意代码，skill 能指挥模型做任何事（包括运行可执行文件）。安装第三方包前务必审阅源码。

## 自我扩展（self-extensible）

这是 pi 最独特的卖点：**内核不是封闭产品**。需要一个命令 / 工具 / provider / 工作流 / UI 调整时，直接让 pi「就地改自己」，`/reload` 后立刻生效，觉得有用就打包分享。官方 50+ 扩展示例里甚至包含「等模型干活时玩 Doom」。`扩展能做的事`包括：自建 sub-agent 与 plan mode、自定义 compaction / 摘要、权限闸门与路径保护、Git checkpoint / 自动提交、SSH 与沙箱执行、把 pi 伪装成 Claude Code 的外观等。

## 多 provider 与订阅复用

15+ 内置 provider（Anthropic、OpenAI、Azure、Bedrock、Google Gemini/Vertex、Mistral、Groq、Cerebras、xAI、OpenRouter、Hugging Face、Kimi For Coding、MiniMax、Ollama……），每个 provider 维护一份「支持工具调用」的模型清单并随版本更新。可用 **API Key 或 OAuth 订阅**鉴权——直连 **Claude Pro/Max、ChatGPT Plus/Pro（Codex）、GitHub Copilot** 订阅，默认不产生额外 API 费用。`/model` 或 `Ctrl+L` 中途换模型，`Ctrl+P` 在常用模型间循环；缺省 provider 经 `models.json` 自定义。

## 会话：树状、可分支、可分享

会话以 JSONL **树**结构存储（每条目带 `id`/`parentId`），所有分支都在同一文件里：

- `/tree` 跳到历史任意节点继续、在分支间切换；可按消息类型过滤、给条目打书签
- `/fork` 从某条历史用户消息新建会话；`/clone` 复制当前活动分支
- `/export` 导出 HTML；`/share` 上传为私有 GitHub gist 并给出可渲染的分享链接

## 上下文工程（context engineering）

极简 system prompt + 可扩展性让用户真正掌控上下文窗口：

- **AGENTS.md**（兼容 `CLAUDE.md`）：启动时从 `~/.pi/agent/`、各级父目录、当前目录加载项目指令
- **SYSTEM.md / APPEND_SYSTEM.md**：按项目替换或追加默认系统提示
- **Compaction**：接近上限时自动摘要旧消息，可用扩展改成按主题压缩、代码感知摘要或换模型
- **Dynamic context**：扩展可在每轮前注入消息、过滤历史、实现 RAG 或长期记忆

## 四种运行模式

| 模式 | 用途 |
|------|------|
| Interactive | 完整 TUI 体验 |
| Print / JSON | `pi -p "query"` 脚本化，`--mode json` 输出事件流 |
| RPC | stdin/stdout 上的 JSON 协议，供非 Node 集成 |
| SDK | 嵌入自己的应用（`@earendil-works/pi-coding-agent` 导出 `createAgentSession` 等）；OpenClaw 是真实集成案例 |

## 安装与常用

```bash
npm install -g --ignore-scripts @earendil-works/pi-coding-agent
# 或
curl -fsSL https://pi.dev/install.sh | sh

export ANTHROPIC_API_KEY=sk-ant-...
pi                       # 交互模式
pi                       # 然后 /login 用订阅登录
pi -p "Summarize repo"   # 一次性 / 脚本
pi --mode rpc            # 进程集成
pi install npm:@foo/pi-tools
pi update --self         # 仅更新 pi 自身
```

默认内置工具：`read` / `bash` / `edit` / `write` / `grep` / `find` / `ls`；可用 `--tools` / `--exclude-tools` 精确放行或禁用以构造只读等模式。

## 供应链加固（少见的工程化亮点）

pi 明确「把 npm 依赖变更当作需审查的代码变更」：直接外部依赖锁定精确版本、`.npmrc` 设 `min-release-age=2` 规避当日发布、`package-lock.json` 为唯一真相且预提交拦截误改、发布前生成并校验 shrinkwrap、安装尽量 `--ignore-scripts`、CI `npm ci --ignore-scripts` + 定时 `npm audit`。对一个 0.x 的开源 Agent 而言，这套防供应链投毒的工程化相当超前。

## 与本知识库的关系

- 与 [[hermes-agent]] **最像**：都是终端 Agent、模型无关、走 [agentskills.io](https://agentskills.io) 的 skill 标准、都鼓励**公开自己的 OSS 会话**用于改进模型（pi 用 `pi-share-hf` 发布到 Hugging Face）。区别：Hermes 偏「自我改进闭环 + 多平台消息网关 + 远程持久环境」，pi 偏「极简内核 + TypeScript 扩展 + 内核可自改」
- 与 [[agent-skills]]：pi 的 skill 是该开放标准的又一实现，印证「description 决定触发、正文承载知识、渐进披露」的设计
- 与 [[trellis]]：Trellis 把 pi（列作 "Pi Agent"）当作支持的 14 个 Agent 之一；二者理念相反——Trellis **强约束**工作流（hook + skill + sub-agent 三层），pi **拆掉一切**只留原语——但可叠加：用 pi 扩展去实现 Trellis 那套 hook / sub-agent
- 与 [[openspec]]：openspec 提供「plan mode 之外的规划层」，pi 干脆不内置 plan mode，正好可由 openspec 这类外部规划层补位
- 与 [[llm-knowledge-bases]]：`AGENTS.md` / `SYSTEM.md` / skill / prompt 模板都是「文件驱动、可 git 协作的持久上下文」的体现

## 风险与局限

- **极简即门槛**：开箱缺 plan mode / sub-agent / todo / MCP，新手容易觉得「太裸」；其价值要在你愿意写或装扩展之后才显现
- **扩展 = 任意代码执行**：pi package 以完整系统权限运行，装第三方包前务必读源码（官方明确警告）
- **无 MCP 生态红利**：想用 MCP 得自己装扩展，享受不到现成 MCP server 的即插即用
- **社区治理较硬**：新贡献者的 issue / PR **默认自动关闭**（维护者每日人工复查），降低了外部参与的即时性
- **高速迭代的 0.x**：命令、设置、SDK 形态仍在演进，版本节奏很快

## 相关链接

- 官网：[pi.dev](https://pi.dev/)，文档 [pi.dev/docs/latest](https://pi.dev/docs/latest)
- GitHub：[earendil-works/pi](https://github.com/earendil-works/pi)（聊天自动化 [earendil-works/pi-chat](https://github.com/earendil-works/pi-chat)）
- 作者博文：[mariozechner.at](https://mariozechner.at/)（pi 设计理念、「你可能不需要 MCP」）
- OSS 会话数据集：[badlogicgames/pi-mono on Hugging Face](https://huggingface.co/datasets/badlogicgames/pi-mono)
- 同类对照：[[hermes-agent]]、[[trellis]]、[[openspec]]、[[agent-skills]]、[[llm-knowledge-bases]]
