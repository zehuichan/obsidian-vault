---
tags: [harness, long-running-agents, claude-agent-sdk, anthropic, workflow]
created: 2026-11-17
source: "raw/2026-11-17-anthropic-effective-harnesses-long-running-agents.md"
---

# Effective Harnesses for Long-Running Agents

让 Agent 跨多个 context window 持续取得进展的 harness 做法：第一个会话用 initializer agent 搭环境，之后每个会话由 coding agent 一次只做一个功能，并用 git 与进度文件给下一个会话留下清晰交接物（Justin Young, Anthropic, 2025-11-26）。

## 核心概念

- **核心难题**：长任务必须分成多个离散会话，每个新会话都没有上一会话的记忆，就像轮班工程师每次上岗都失忆
- **compaction 不够**：即便 Opus 4.5 在 Claude Agent SDK 里跨窗口循环，只给「做一个 claude.ai 克隆」这类高层 prompt，也做不出生产级应用
- **两种典型失败**：① 想一次性做完（one-shot），上下文在实现中途耗尽，留下半成品且没记录；② 项目后期，新实例看到已有进展就宣布完工
- **两部分方案**：initializer agent 搭环境；coding agent 增量推进并保持环境「干净」（达到可合并主干的状态）
- 两个「Agent」仅初始 prompt 不同，system prompt、工具和整体 harness 完全一致

## 环境管理

**交接物（initializer 产出）**
- `init.sh`：启动开发服务器
- `claude-progress.txt`：记录各 Agent 做过的事
- 初始 git commit：展示新增了哪些文件

**功能清单（feature list）**
- 把用户的初始 prompt 展开成完整的功能需求文件；claude.ai 克隆例子里超过 200 个功能，全部初始标记为 failing（`"passes": false`），让后来的 Agent 清楚「完整功能长什么样」
- 每条含 `category`、`description`、`steps`、`passes`
- 约束：只允许修改 `passes` 字段，并用强措辞如「删除或编辑测试是不可接受的」
- 用 JSON 而不是 Markdown：模型更不容易不当改写 JSON 文件

**增量推进**
- 一次只做一个功能，这是解决「一次做太多」的关键
- 每次改动后让环境保持干净：用描述性 commit message 提交 git，并在进度文件里写摘要；这样可以用 git 回退坏改动、恢复可用状态

**测试**
- Claude 倾向未经充分测试就把功能标成完成，即使跑了单元测试或对开发服务器 `curl` 也看不出端到端不工作
- 明确要求用浏览器自动化工具（如 Puppeteer MCP）并像人类用户那样测试后，端到端验证大幅改善，能发现代码里看不出的 bug
- 局限：视觉能力和浏览器自动化的限制；例如 Puppeteer MCP 看不到浏览器原生 alert 弹窗，依赖它们的功能 bug 更多

## 会话开场流程（Getting up to speed）

1. `pwd` 确认工作目录，只能编辑该目录下的文件
2. 读 git 日志和进度文件，了解最近做了什么
3. 读功能清单，选择未完成的最高优先级功能

然后让 initializer 写好的 `init.sh` 启动开发服务器，先跑一次基础端到端测试（claude.ai 例子里是新建聊天、发消息、收回复），确认应用没被上个会话留在损坏状态，再开始新功能；否则新功能只会让问题更糟。

## 失败模式与对策

| 问题 | Initializer 行为 | Coding Agent 行为 |
| --- | --- | --- |
| 过早宣布整个项目完成 | 建立 JSON 功能清单 | 会话开始读清单，只选一个功能做 |
| 环境留下 bug 或未记录的进展 | 写初始 git 仓库和进度笔记 | 开场读进度和 git log，跑基础测试；收尾 commit 并更新进度 |
| 过早把功能标成完成 | 建立功能清单 | 自我验证，仔细测试后才标记 passing |
| 每次要花时间摸索怎么跑应用 | 写 `init.sh` | 开场读 `init.sh` |

## 未解问题

- 单一通用 coding agent 是否最优，还是多 Agent 架构（测试 Agent、QA Agent、代码清理 Agent 等）更好
- 目前针对全栈 Web 开发优化，能否推广到科研、金融建模等长时任务

## 相关链接

- [[harness-design-long-running-apps]] — 在本文基础上的后续：planner / generator / evaluator 三 Agent 与简化过程
- [[effective-context-engineering]] — compaction、笔记、sub-agent 的整体框架
- [[agent-memory-components]] — 进度文件这类笔记如何发展成记忆体系
- [[trellis]] — 把 spec / task / journal 固化进仓库，是同一思路的团队级实现
- [[pi]] — 另一种极简 harness 取舍
- [[anthropic-learning-path]] — Anthropic 官方学习路径总览

## 参考

- 原文: https://www.anthropic.com/research/effective-harnesses-for-long-running-agents
