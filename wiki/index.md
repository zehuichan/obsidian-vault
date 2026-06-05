# Wiki 索引

> 此文件由 LLM 维护，按主题分类列出所有 wiki 文章。

## 目录

### LLM & AI

- [[llm-knowledge-bases]] — 利用 LLM 增量编译持久 wiki 替代传统 RAG 的知识管理模式（Karpathy, 2026）
- [[open-design]] — 把已有编码 Agent 转化为设计引擎的开源工作流，由 31 个 skills + 72 个 design systems 驱动（nexu-io, 2026）
- [[hermes-agent]] — Nous Research 开源的自我改进 AI Agent，具备闭环学习、跨平台接入和 skill 自动创建能力
- [[agent-skills]] — Agent Skill 的规范结构、渐进式加载、触发机制、构建方法和常见设计模式
- [[impeccable]] — 把 23 条命令 + 27 条反模式检测 + brand/product 双 register 编入 AI 编码助手的设计 Skill 包（Paul Bakaus, 2026）
- [[openspec]] — 轻量级 spec-driven 框架：v1.3 引入 `/opsx:propose|apply|archive` artifact-guided 工作流 + Dashboard + profile 系统，仓库内活规格与变更提案包，跨 25+ Agent 的规划层（Fission AI, 2026）
- [[codegraph]] — 本地预索引代码知识图谱 MCP 服务，为 Claude Code/Cursor 等 Agent 提供符号搜索与调用链分析，平均节省 35% 成本（colbymchenry, 2026）
- [[trellis]] — 团队级 AI 编码 Agent 脚手架 + 仓库内 LLM wiki：spec / task / workflow / journal 全部固化为 git Markdown，按平台用 hook、auto-skill、sub-agent 注入到 14 个编码 Agent（Mindfold HQ, 2026）
- [[pi]] — 极简、可自我扩展的终端编码 Agent harness：内核只留 4 个工具 + 极简 prompt，刻意不内置 sub-agent / plan mode / MCP，全部交给 TypeScript 扩展、skill、prompt 模板与 pi package（Mario Zechner / earendil-works, 2026）

### Web 框架 & 前端

- [[ilha]] — 极小、同构的岛屿架构（islands architecture）Web UI 库：signal 细粒度响应式、无虚拟 DOM/无编译器，SSR + 零闪烁 hydration，后端无关（Nitro/Hono），源码小到可塞进 AI 上下文窗口（ilhajs, 2026）
