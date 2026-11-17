---
tags: [anthropic, claude-code, agent-engineering, learning-path, harness, context-engineering]
created: 2026-11-17
source: "raw/2026-11-17-anthropic-engineering-index.md, raw/2026-11-17-claude-code-docs-overview.md, raw/2026-11-17-claude-blog-articles.md, raw/2026-11-17-anthropic-news-index.md"
---

# Anthropic 官方学习路径（追踪最新 Agent 范式）

想跟上 Claude Code / Agent 的最新做法，不必泛读 Anthropic 全站：按 **Engineering → Claude Code 文档 → Claude 博客 → News** 的顺序读，前者给范式，后者给落地与信号。

## 核心概念

- **范式首发在 Engineering**：harness、上下文工程、记忆、评估、沙箱与权限这些方法论，都以工程文章的形式先出现；文章实际挂在 `anthropic.com/research/<slug>` 路径下，但列表页在 `/engineering`
- **落地看 Claude Code 文档**：范式最终会变成 `CLAUDE.md`、skills、hooks、子 Agent、定时任务、Agent SDK 这些具体功能
- **博客是实践指南，News 只扫信号**：产品发布类文章可略读；News 里与范式相关的只有 Agent Skills 开放标准和模型发布节奏
- **精读状态**：下列带 `→ [[...]]` 的 6 篇已全文剪藏并单独编译成 wiki 文章；其余各篇的主题归类仅依据标题与列表摘要，尚未精读，精读后应单独剪藏并补充细节

## 1. Engineering：范式源头（共 24 篇，截至 2026-11-17）

入口：<https://www.anthropic.com/engineering>。按主题分组如下（日期为发布日）。

**Harness 与长时运行 Agent**
- Effective harnesses for long-running agents（2025-11-26）→ [[effective-harnesses-long-running-agents]]
- Harness design for long-running application development（2026-03-24）→ [[harness-design-long-running-apps]]
- Building a C compiler with a team of parallel Claudes（2026-02-05）
- Scaling Managed Agents: Decoupling the brain from the hands（2026-04-08）
- How we built our multi-agent research system（2025-06-13）

**上下文、记忆与工具**
- Effective context engineering for AI agents（2025-09-29）→ [[effective-context-engineering]]
- The components of agent memory（2026-11-06）→ [[agent-memory-components]]：Claude 要从经验中学习，必须在还不知道下一步是什么时就决定什么值得记住；文章讲 "dreaming" 如何帮助它整理记忆、纠正错误
- Writing effective tools for agents — with agents（2025-09-11）
- Code execution with MCP: Building more efficient agents（2025-11-04）
- Introducing advanced tool use on the Claude Developer Platform（2025-11-24）
- Equipping agents for the real world with Agent Skills（2025-10-16）→ [[equipping-agents-with-agent-skills]]
- Desktop Extensions: One-click MCP server installation for Claude Desktop（2025-06-26）

**评估**
- Demystifying evals for AI agents（2026-01-09）→ [[demystifying-agent-evals]]
- Quantifying infrastructure noise in agentic coding evals（2026-02-05）
- Eval awareness in Claude Opus 4.6's BrowseComp performance（2026-03-06）
- Designing AI-resistant technical evaluations（2026-01-21）
- Building strong safety evaluations（2026-11-13）

**安全、沙箱与权限**
- Beyond permission prompts: making Claude Code more secure and autonomous（2025-10-20）
- How we built Claude Code auto mode: a safer way to skip permissions（2026-03-25）
- How we contain Claude across products（2026-05-25）
- Confidential Inference via Trusted Virtual Machines（2025-06-18）

**工程运营**
- A postmortem of three recent issues（2025-09-17）
- An update on recent Claude Code quality reports（2026-04-23）
- Redesigning source control at Anthropic（2026-10-13）

**精读顺序与进度**（按与本库主题的契合度挑选，6 篇均已读完并编译）：
1. [[effective-context-engineering]] — 总框架：上下文是有限资源
2. [[effective-harnesses-long-running-agents]] → [[harness-design-long-running-apps]] — 长时任务 harness 从 initializer + coding agent 演进到 planner / generator / evaluator
3. [[agent-memory-components]] — 记忆与 dreaming
4. [[equipping-agents-with-agent-skills]] — 与 [[agent-skills]] 直接对应
5. [[demystifying-agent-evals]] — 评估方法，也是前面几篇调教评估器的通用背景

下一批可考虑：Building strong safety evaluations、How we built our multi-agent research system、Writing effective tools for agents — with agents、Code execution with MCP、How we built Claude Code auto mode。

## 2. Claude Code 文档：把范式落到功能

入口：<https://code.claude.com/docs>，完整页面索引在 <https://code.claude.com/docs/llms.txt>，适合批量剪藏。概览页列出的与范式相关的能力：

- **指令与记忆**：项目根目录的 `CLAUDE.md` 每次会话开头读取；仓库已有 `AGENTS.md` 时，Claude Code 可以直接读它代替 `CLAUDE.md`；另有自动记忆，把学到的东西跨会话保存
- **Skills 与 Hooks**：Skills 打包可复用工作流（如 `/review-pr`）；Hooks 在 Claude Code 动作前后执行 shell 命令（如每次改文件后自动格式化）
- **MCP**：用开放标准把 Claude Code 接到 Google Drive、Jira、Slack 或自定义工具
- **并行与自定义 Agent**：主 Agent 协调多个子 Agent；后台 Agent 可并行跑多个完整会话；Agent SDK 用于自建 Agent，完全控制编排、工具权限
- **CLI 可组合**：`claude -p` 支持管道和 CI 用法
- **定时任务**：云端定时任务（原 routines，电脑关机也运行，可由 API 调用或 GitHub 事件触发，用 `/schedule` 创建）、桌面定时任务、会话内 `/loop`
- **跨端迁移**：Remote Control、Dispatch、`claude --teleport`、`/desktop`、Slack 里 `@Claude`
- **延伸阅读**：Claude Academy 免费课程（含 Claude Code 101）、"A harness for every task"（Claude Code 团队如何用动态工作流同时编排多个子 Agent）、Common workflows and best practices

## 3. Claude 博客：实践指南

入口：<https://claude.com/blog>。与范式相关的近期文章：

- **Building always-on agents**（2026-11-13）：常驻 Agent 持续运行、跨多个会话决定何时如何行动，文章讲其核心原语及组合方式
- **Operate Claude Managed Agents inside your security perimeter**（2026-11-12）：自托管 Managed Agents，Agent、凭据和会话历史由自己托管，并可选择请求发往哪里
- **Using Claude Opus 5.5 as your daily driver for knowledge work**（2026-11-12）：Opus 5.5 现在是 Pro / Max / Ultra / Team 计划的默认模型，文章对比各模型并给出使用建议

其余为产品发布（Whiteboards / Sheets / Data、Meetings、Claude Tag、嵌入式应用、行业插件、客户案例），按需略读。

## 4. News：只扫关键信号

入口：<https://www.anthropic.com/news>。IPO、政策、合作伙伴类内容可跳过，只看两类：

- **Agent Skills 开放标准**（2026-10-22）：Anthropic 把 Agent Skills 捐给 Agentic AI Foundation；自 2025 年 12 月发布标准以来，已有 70 多个 Agent 平台集成，包括 ChatGPT、Cursor、GitHub Copilot、VS Code
- **模型发布节奏**：Opus 5.5（09-22）、Sonnet 5.5（09-28）、Haiku 5.5（10-07，最快的小模型，面向摘要、子 Agent、浏览器使用等高并发场景）、Fable 5.2 与 Mythos 5.2（11-09，面向高难度技术工作）

## 与本库已有内容的关系

- 范式层的 Skills：[[agent-skills]] 讲规范与渐进式加载，[[equipping-agents-with-agent-skills]] 是官方原文，[[addy-osmani-agent-skills]]、[[matt-pocock-skills]] 是具体的 Skill 包，可与 Engineering 里的 Agent Skills 文章对照
- harness 与脚手架：[[pi]] 是极简 harness 的另一种取舍，[[trellis]] 把 spec / task / workflow 固化进仓库并按平台注入 hook 与子 Agent，可对照 "Harness design" 一类文章
- 记忆与知识：[[llm-knowledge-bases]] 与 "The components of agent memory" 处理的是同一类问题
- 上下文与工具：[[headroom]]（上下文压缩）、[[codegraph]]（代码图谱 MCP）可对照 context engineering 与 "Code execution with MCP"
- 常驻与主动 Agent：[[dsh-superagent]] 与 "Building always-on agents" 的方向相近
- 评估：[[reviewbench]] 可对照 "Demystifying evals for AI agents"

## 维护建议

- 每次更新时只检查 Engineering 列表是否有新文章，再决定是否剪藏
- 精读时逐篇剪藏到 `raw/`，每篇单独编译成 wiki 文章，并回填到本文对应分组
- 文档部分按主题选页剪藏，不要整站抓取

## 相关链接

- [[agent-skills]] — Skill 规范、渐进式加载与设计模式
- [[trellis]] — 仓库内 LLM wiki 与多平台 Agent 脚手架
- [[pi]] — 极简终端编码 Agent harness
- [[llm-knowledge-bases]] — 用 LLM 增量编译持久 wiki
- [[dsh-superagent]] — 主动式编码 Agent 原型
