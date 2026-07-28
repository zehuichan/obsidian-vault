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
- [[headroom]] — 面向 AI Agent 的本地优先上下文压缩层，通过 library / proxy / wrapper / MCP 压缩工具输出、日志、RAG、文件和历史对话，宣称节省 47%–92% 输入 token（Headroom Labs, 2026）
- [[trellis]] — 团队级 AI 编码 Agent 脚手架 + 仓库内 LLM wiki：spec / task / workflow / journal 全部固化为 git Markdown，按平台用 hook、auto-skill、sub-agent 注入到 14 个编码 Agent（Mindfold HQ, 2026）
- [[pi]] — 极简、可自我扩展的终端编码 Agent harness：内核只留 4 个工具 + 极简 prompt，刻意不内置 sub-agent / plan mode / MCP，全部交给 TypeScript 扩展、skill、prompt 模板与 pi package（Mario Zechner / earendil-works, 2026）
- [[ponytail]] — 让 AI 编码 Agent 像 senior dev 一样写最少可用代码的规则集/插件：七级决策阶梯（YAGNI → 复用 → stdlib → 原生 → 已装依赖 → 一行 → 最小 diff），宣称 54% 更少代码且 100% 保留安全维度（DietrichGebert, 2026）
- [[agentic-erp-landing-cases]] — 全球 Agentic ERP 落地案例深度解析：三大阵营（传统巨头/云原生/AI Native+国产）厂商战略、跨行业 ROI 数据与四大落地壁垒（王吉伟, 2026）
- [[how-to-build-agentic-erp]] — 在已有 ERP 上搭建 Agentic 能力的实操指南：五层技术栈、成熟度四级爬坡、四大前提、三条切入路径，含自研 ERP 的 MCP 工具设计与第一个落地场景
- [[mastra]] — 现代 TypeScript AI 应用/Agent 框架：模型路由、Agents、图工作流、HITL、记忆/RAG、MCP、evals/observability，可嵌 React/Next/Node 或独立部署（YC W25）
- [[opencode]] — 开源 AI 编码 Agent：终端 / 桌面 / IDE，LSP + 多会话 + 会话分享，75+ 模型提供商，可复用 Copilot / ChatGPT 订阅，隐私优先不存储代码上下文（Anomaly, 2026）
- [[matt-pocock-skills]] — 「Skills For Real Engineers」：可组合的 Agent Skills，用 grilling / CONTEXT.md / TDD / deep modules 修复对齐、啰嗦、无反馈与泥球架构（Matt Pocock, 2026）
- [[grok-build]] — SpaceXAI 开源终端 AI 编码 Agent（`grok`）：Rust TUI + agent runtime，支持交互/headless/ACP，含 MCP、skills、hooks、sandbox（xai-org, 2026）
- [[agent-browser]] — 面向 AI Agent 的浏览器自动化 CLI：Rust + CDP daemon、snapshot refs、MCP/skills、会话认证与云浏览器 provider（Vercel Labs, 2026）

### DevOps & 包管理

- [[sentry]] — 开发者优先的错误追踪与 APM 平台：Python/Django 后端 + React 前端 monorepo，20+ 语言 SDK，Fair Source 许可，支持 SaaS 与 self-hosted（getsentry, 2026）
- [[verdaccio]] — 轻量级 Node.js 私有 npm 代理 registry：零配置本地运行、代理缓存 npmjs.org、多 registry 聚合、覆盖公共包，支持 Docker / Helm 部署（Verdaccio Community, 2026）

### Web 框架 & 前端

- [[codemirror]] — 用于 Web 的可嵌入代码编辑器组件，提供语法高亮、补全、折叠、搜索替换、解析、协作编辑、主题和扩展接口等能力
- [[ilha]] — 极小、同构的岛屿架构（islands architecture）Web UI 库：signal 细粒度响应式、无虚拟 DOM/无编译器，SSR + 零闪烁 hydration，后端无关（Nitro/Hono），源码小到可塞进 AI 上下文窗口（ilhajs, 2026）
- [[openapi-ts-request]] — 从 Swagger2/OpenAPI3/Apifox 生成 TS/JS 类型、任意客户端请求函数、mock、枚举翻译、react-query/vue-query 与 JSON Schemas（openapi-ui, 2026）

### 媒体 & 创作工具

- [[opencut]] — 开源 CapCut 替代品：Web/桌面/移动视频编辑器；classic 可用，main 正以 Rust core + plugin-first + MCP 从零重写（OpenCut-app, 2026）
