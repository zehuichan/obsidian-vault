---
tags: [llm, ai-agents, context-compression, mcp, tooling]
created: 2026-06-25
source: "raw/2026-06-25-headroom.md"
---

# Headroom

Headroom 是一个面向 AI Agent 的本地优先上下文压缩层，在工具输出、日志、RAG chunk、文件和对话历史进入 LLM 前进行压缩，并通过可逆缓存让模型在需要时取回原文。

## 核心概念

- **压缩入口多形态**：可作为 Python/TypeScript library、OpenAI-compatible proxy、Agent wrapper 或 MCP server 接入
- **目标是降上下文成本**：README 宣称真实 Agent workload 可减少 47% 到 92% 输入 token，同时保持标准 benchmark 的准确率
- **可逆压缩 CCR**：原文保存在本地缓存中，模型可通过 `headroom_retrieve` 按需取回
- **内容路由**：`ContentRouter` 判断输入类型，再分派给 JSON、代码 AST、文本模型等压缩器
- **跨 Agent 记忆**：在 Claude、Codex、Gemini 等工具之间共享压缩后的上下文和记忆
- **输出 token 也可控**：proxy 可通过 verbosity steering 与 effort routing 减少模型回复中的礼貌套话、重复代码和不必要深思考

## 详细内容

### 解决的问题

Agent 系统的上下文消耗主要来自工具输出、日志、代码搜索结果、RAG 片段和长对话历史。传统做法通常依赖模型供应商的原生 compaction，或在应用层手写摘要逻辑；Headroom 的定位是提供一个横跨 Agent、应用框架和 MCP 客户端的统一压缩层。

它的典型收益场景包括：

| 场景 | README 给出的节省 |
|------|------------------|
| Code search 100 results | 92% |
| SRE incident debugging | 92% |
| GitHub issue triage | 73% |
| Codebase exploration | 47% |

### 架构

Headroom 的核心链路是：

```text
Agent / App
  -> Headroom
  -> CacheAligner
  -> ContentRouter
  -> SmartCrusher / CodeCompressor / Kompress-base
  -> CCR local cache
  -> LLM provider
```

关键组件：

- `CacheAligner` 稳定 prompt 前缀，提高 Anthropic/OpenAI 等供应商 KV cache 命中率
- `ContentRouter` 根据内容类型选择压缩器
- `SmartCrusher` 面向 JSON、数组、嵌套对象等结构化数据
- `CodeCompressor` 面向 Python、JS、Go、Rust、Java、C++ 等代码，使用 AST 感知压缩
- `Kompress-base` 是基于 HuggingFace 的文本压缩模型，训练于 agentic traces
- `CCR` 保存原文，支持通过 MCP 工具按需 retrieval

这与 [[codegraph]] 的思路有交集：二者都试图让 Agent 不必把大量原始上下文直接塞进 prompt。差别在于 CodeGraph 是预索引的符号知识图谱，偏代码结构检索；Headroom 是通用上下文压缩层，覆盖日志、RAG、工具输出、文件和历史对话。

### 接入方式

Headroom 支持四类主要接入：

| 方式 | 用法 | 适合场景 |
|------|------|----------|
| Library | `compress(messages)` | 自己控制应用代码 |
| Proxy | `headroom proxy --port 8787` | 想以 OpenAI-compatible API 零改造接入 |
| Agent wrap | `headroom wrap claude/codex/cursor/aider/...` | 给现有 coding agent 加一层本地压缩 |
| MCP server | `headroom_compress`, `headroom_retrieve`, `headroom_stats` | MCP-native Agent 或 IDE |

它还提供 `headroom learn`，用于从失败会话中挖掘修正建议，并写入 `CLAUDE.md`、`AGENTS.md`、`GEMINI.md` 等项目级指导文件。这个能力与 [[agent-skills]]、[[trellis]] 里把经验固化为可复用规则/skill 的模式相近。

### 何时适合使用

Headroom 最适合长期、高频使用 Agent 的工作流，尤其是：

- 每天运行 AI coding agent，且工具输出、代码搜索、日志分析消耗大量 token
- 同时使用 Claude、Codex、Cursor、Aider、Copilot CLI 等多个 Agent，希望共享上下文记忆
- 需要本地优先与可逆压缩，不想把压缩任务交给外部托管 API
- 有 MCP 客户端或 proxy 层，能接受在 Agent 与 LLM provider 之间插入一个本地服务

不适合的情况：

- 只使用单一供应商的原生上下文管理，且 token 成本不是问题
- 环境无法运行本地 proxy、wrapper 或额外进程
- 对压缩引入的可观测性、缓存 TTL、检索失败路径还没有治理要求

### 对 Agent 系统设计的启发

Headroom 代表一种「上下文预算层」思路：不要只在 prompt engineering 阶段思考上下文，而是把上下文压缩、缓存命中、按需取回、跨 Agent 记忆做成基础设施。

在更大的 Agent 系统里，它可以位于 [[pi]] 这类 agent harness、[[trellis]] 这类团队级 agent 工作流，或 [[how-to-build-agentic-erp]] 这类企业业务 Agent 的工具层前后，负责降低日志、报表、RAG 和工具调用结果进入模型前的 token 压力。

## 相关链接

- [[codegraph]] — 本地代码知识图谱，与 Headroom 互补地减少原始代码上下文输入
- [[agent-skills]] — 将 Agent 能力和经验封装为可发现、可调用的标准单元
- [[trellis]] — 团队级 AI 编码 Agent 脚手架与仓库内 LLM wiki
- [[pi]] — 极简终端编码 Agent harness，可对比 Headroom 的 proxy/wrapper 思路
- [[how-to-build-agentic-erp]] — 企业 Agent 场景中的工具层、RAG 与治理设计
