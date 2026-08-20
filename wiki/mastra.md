---
tags: [ai-framework, typescript, agents, workflows, mcp, ycombinator]
created: 2026-07-17
source: "https://github.com/mastra-ai/mastra"
---

# Mastra

面向现代 TypeScript 栈的 AI 应用与 Agent 框架：从原型到生产一体提供模型路由、Agent、图工作流、人机协同、记忆/RAG、MCP、评测与可观测性，可嵌入 React / Next.js / Node，也可独立部署。

## 核心概念

- **Model routing**：统一接口连接 40+ 供应商（OpenAI、Anthropic、Gemini 等），模型目录见 [mastra.ai/models](https://mastra.ai/models)
- **Agents**：自主 Agent 用 LLM + Tools 求解开放任务，内部迭代直到产出最终答案或触发停止条件
- **Workflows**：图工作流引擎，用 `.then()` / `.branch()` / `.parallel()` 编排多步流程，适合需要显式控制执行路径的场景
- **Human-in-the-loop**：可挂起 Agent/工作流等待用户输入或审批；执行状态落 storage，可长期暂停后原位恢复
- **Context management**：对话历史、RAG 检索，以及 Observational Memory，让 Agent 在正确时机拿到正确上下文
- **MCP servers**：把 Agent、Tools 与结构化资源暴露为 MCP，供任意支持协议的系统调用
- **Production essentials**：内置 evals 与 observability，支撑持续观察、度量与迭代

## 详细内容

### 定位与集成

Mastra 的目标是「用一套 TypeScript 框架把 AI 产品从早期原型推到可上线」。可嵌入现有 React、Next.js、Node 应用，也可作为独立服务部署。前端可与 Vercel AI SDK UI、CopilotKit 等 agentic UI 库对接。项目为 YC W25，仓库约 26k+ stars，语言几乎全是 TypeScript。

### 快速上手

推荐用脚手架创建项目：

```bash
npm create mastra@latest
```

创建后启动开发服务器；Mastra Studio（构建/测试/管理 agents、workflows、tools 的 UI）默认在 `http://localhost:4111`。

官方还提供给编码 Agent 用的预置 prompt：按项目名与 LLM provider（openai / anthropic / groq / google / cerebras / mistral）问答后执行 `npm create mastra@latest`，再启动 Studio。

### 仓库结构（概览）

Monorepo 主要目录：`packages/`（核心包）、`docs/`、`examples/`、`templates/`、`client-sdks/`、`agent-sdks/`、`server-adapters/`、`deployers/`、`stores/`、`workflows/`、`voice/`、`observability/`、`integrations/`，以及企业特性目录 `ee/`。

### 许可

双许可：

- **Apache-2.0**：核心框架与绝大多数代码
- **Mastra Enterprise License**：任意名为 `ee/` 的目录（源码可见）；生产使用需企业许可，开发/测试可免费使用

安全问题请发至 security@mastra.ai。

## 相关链接

- [[hermes-agent]] — 同为 Agent 运行时，但偏终端 CLI + 多平台网关 + 闭环学习；Mastra 偏 TypeScript 应用框架 + 工作流/Studio/部署
- [[agent-skills]] — Agent Skill 规范；Mastra 侧能力以 Agents / Tools / MCP / Workflows 为主，可对照 skill 渐进加载思路
- [[pi]] — 极简终端 Agent harness；Mastra 是应用侧全栈框架，职责与集成面更大
- [[headroom]] — Agent 上下文压缩层；可与 Mastra 的 context / memory / RAG 能力互补思考
- [[vercel-ai-gateway]] — Vercel 统一模型网关；Mastra 自带 model routing，应用侧也可把 LLM 调用接到 Gateway / AI SDK

## 参考

- GitHub: https://github.com/mastra-ai/mastra
- 官网 / 文档: https://mastra.ai · https://mastra.ai/docs
- 安装指南: https://mastra.ai/docs/getting-started/installation
- Build with AI: https://mastra.ai/docs/getting-started/build-with-ai
- Discord: https://discord.gg/BTYqqHKUrf
- 原始剪藏: `raw/2026-07-17-mastra-typescript-ai-framework.md`
