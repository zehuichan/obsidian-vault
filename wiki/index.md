# Wiki 索引

> 此文件由 LLM 维护，按主题分类列出所有 wiki 文章。

## 目录

### LLM & AI

- [[llm-knowledge-bases]] — 利用 LLM 增量编译持久 wiki 替代传统 RAG 的知识管理模式（Karpathy, 2026）
- [[open-design]] — 把已有编码 Agent 转化为设计引擎的开源工作流，由 31 个 skills + 72 个 design systems 驱动（nexu-io, 2026）
- [[hermes-agent]] — Nous Research 开源的自我改进 AI Agent，具备闭环学习、跨平台接入和 skill 自动创建能力
- [[agent-skills]] — Agent Skill 的规范结构、渐进式加载、触发机制、构建方法和常见设计模式
- [[agency-agents]] — 230+ 人格化专业 Agent 花名册（The Agency），可安装到 Claude Code / Cursor / Codex / OpenCode 等（msitarzewski, 2026）
- [[impeccable]] — 把 23 条命令 + 27 条反模式检测 + brand/product 双 register 编入 AI 编码助手的设计 Skill 包（Paul Bakaus, 2026）
- [[diagram-design]] — 39 种编辑级 HTML/SVG 图的 Agent Skill：从网站抽品牌、重绘 draw.io/Mermaid、无阴影无 Mermaid slop（Cathryn Lavery, 2026）
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
- [[addy-osmani-agent-skills]] — 生产级工程技能包：25 个覆盖 Define→Ship 的 skill、9 条阶段命令、4 个评审人格、反合理化表与仓库内三层 eval（Addy Osmani, 2026）
- [[grok-build]] — SpaceXAI 开源终端 AI 编码 Agent（`grok`）：Rust TUI + agent runtime，支持交互/headless/ACP，含 MCP、skills、hooks、sandbox（xai-org, 2026）
- [[agent-browser]] — 面向 AI Agent 的浏览器自动化 CLI：Rust + CDP daemon、snapshot refs、MCP/skills、会话认证与云浏览器 provider（Vercel Labs, 2026）
- [[e2e-testing-framework]] — TesterArmy 开源的 Web/移动端 E2E 框架：自然语言目标与 locator 断言写在同一条测试里，验证通过的 `agent.act` 可回放且不再调模型（tester-army, 2026）
- [[cua]] — computer-use 2.0 开源栈：后台桌面 Drivers（CLI/MCP）、跨 OS Sandbox、Bench/RL 环境与 Apple Silicon Lume 虚拟化（trycua, 2026）
- [[lean4-skills]] — 面向 AI 编码 Agent 的 Lean 4 定理证明工作流包：prove/review/golf 循环、mathlib 搜索、公理检查与 LSP-first 护栏（Cameron Freer, 2025）
- [[reviewbench]] — GitHub 开源的 AI 代码审查离线基准：219 个真实 PR 的人工复核 golden set，用 grounded / augmented precision 与 recall 评估审查 Agent（review-bench, 2026）
- [[neuve]] — 面向 Agent 时代的开源代码评审工具：commit range → batch / review unit，按复杂度×可验证性分入 Automation / Agentic / Human Suggested / Human Required 四条 lane；四个 finder + 证据再核实、本机 review desk、`.git/neuve/` append-only SQLite、可走 Anthropic / ChatGPT / Claude 订阅三种路线（NeuveAI, 2026）
- [[vercel-ai-gateway]] — Vercel 统一 AI API：一把密钥访问数百个模型（文本/图像/视频/语音/embeddings），零加价、自动 failover，兼容 AI SDK / OpenAI / Anthropic（Vercel, 2026）
- [[design-agent]] — LangChain 开源的团队设计 Agent（Content Studio）：基于 Deep Agents + Managed Deep Agents，Slack 里对话出品牌合规的图/deck/图表/短片；每线程独立沙箱、35 个 skills、品牌即代码 + brand-lint 等闸门、异步 helper 与唤醒 harness、密钥不进沙箱（langchain-ai, 2026）
- [[dsh-superagent]] — DeepSeek 公开的研究原型：绑定单个 GitHub 仓库的主动式编码 Agent（DeepSeek Harness bundle），用 Source/Eval/Mandate/Ticket 把「少占人的注意力」做成授权与面板的硬规则；监听 GitHub 与可选的飞书，默认 observe 只记录主动工作（deepseek-ai, 2026）
- [[effective-context-engineering]] — Context 是有限的注意力预算：找最小高信号 token 集合；system prompt 找对高度、精简工具、即时检索与混合策略，长时任务靠 compaction / 结构化笔记 / sub-agent（Anthropic, 2025）
- [[effective-harnesses-long-running-agents]] — 跨多个 context window 的 Agent harness：initializer + coding agent，JSON 功能清单、一次一个功能、`init.sh` / progress 文件 / git 交接、浏览器端到端自测（Justin Young, Anthropic, 2025）
- [[harness-design-long-running-apps]] — GAN 式 generator / evaluator + planner 三 Agent harness：sprint contract、Playwright 评估器、评分标准编码设计品味，及随模型升级逐个拆组件（$9/20 分钟 vs $200/6 小时对照；DAW 运行 $124.70）（Anthropic, 2026）
- [[agent-memory-components]] — Agent 记忆的组成与「dreaming」：CL-Bench 学习闭环、Pokémon 中的显著性判断、让 Claude 自己判断记什么，以及纠错、建 wiki、提升到顶层、聚焦目标、补缺口五种用法（Lance Martin, Anthropic, 2026）
- [[equipping-agents-with-agent-skills]] — Agent Skills 官方介绍：三层渐进式披露、与确定性代码执行结合（PDF Skill 例）、从评估起步的开发建议、安全审计提示与路线图（Anthropic, 2025）
- [[demystifying-agent-evals]] — Agent 评估方法：task / trial / grader / transcript / outcome 术语，代码、模型、人工三类 grader，pass@k 与 pass^k，从 0 到 1 的 8 步路线图及与生产监控、A/B、人工研究的组合（Anthropic, 2026）
- [[anthropic-learning-path]] — 追踪 Claude Code / Agent 最新范式的 Anthropic 官方阅读路径：Engineering（24 篇，按 harness / 上下文与记忆 / 评估 / 安全分组）→ Claude Code 文档 → Claude 博客 → News 信号，含精读顺序与本库对照（Anthropic, 2026）

### 产品开发 & 项目管理

- [[linear]] — 面向「团队与 Agent」的产品开发系统：issue / 计划 / 评审同在一处，`@Linear` 把整条 issue 委托给云端 coding agent 出 Draft PR；Loops 自动化（Slack / GitHub 触发）、Diffs 评审（含栈式 PR、移动端）、BYOK，并提供 MCP 与 llms.txt 给 Agent 接入（Linear, 2026，首页营销页）

### DevOps & 包管理

- [[sentry]] — 开发者优先的错误追踪与 APM 平台：Python/Django 后端 + React 前端 monorepo，20+ 语言 SDK，Fair Source 许可，支持 SaaS 与 self-hosted（getsentry, 2026）
- [[verdaccio]] — 轻量级 Node.js 私有 npm 代理 registry：零配置本地运行、代理缓存 npmjs.org、多 registry 聚合、覆盖公共包，支持 Docker / Helm 部署（Verdaccio Community, 2026）

### 架构 & 数据平台

- [[json-schema-dynamic-data-model]] — 把 JSON Schema 当源语言，编译成可缓存、可灰度、可观测的动态数据模型运行时：JSONB 事实 + 物化列/投影、Outbox、三级缓存与字段级安全（微信公众号, 2026）

### 语言 & 运行时

- [[porffor]] — 从零实现的 100% AOT JS/TS 编译器：编成 Wasm / C / 原生二进制，无解释器与 JIT；自研 2c 做 Wasm→C（CanadaHonk, 2026）

### Web 框架 & 前端

- [[codemirror]] — 用于 Web 的可嵌入代码编辑器组件，提供语法高亮、补全、折叠、搜索替换、解析、协作编辑、主题和扩展接口等能力
- [[ilha]] — 极小、同构的岛屿架构（islands architecture）Web UI 库：signal 细粒度响应式、无虚拟 DOM/无编译器，SSR + 零闪烁 hydration，后端无关（Nitro/Hono），源码小到可塞进 AI 上下文窗口（ilhajs, 2026）
- [[openapi-ts-request]] — 从 Swagger2/OpenAPI3/Apifox 生成 TS/JS 类型、任意客户端请求函数、mock、枚举翻译、react-query/vue-query 与 JSON Schemas（openapi-ui, 2026）
- [[pinia-colada]] — Vue 数据获取层：建在 Pinia 上的 `useQuery` / `useMutation`，带缓存去重、乐观更新、插件与 Nuxt SSR（posva, 2026）
- [[standard-json]] — 基于 Standard Schema 的统一 JSON Schema 转换器：`toJsonSchema` + `loadVendor`，覆盖 Zod/Valibot/ArkType 等（standard-community, 2026）

### 媒体 & 创作工具

- [[pascal-editor]] — 开源本地优先 3D 建筑编辑器：R3F + WebGPU，CLI 管本地 SQLite 项目，MCP + `pascal-3d` / `furniture-fit` skills 让 Agent 读写同一场景（pascalorg, 2026）
- [[opencut]] — 开源 CapCut 替代品：Web/桌面/移动视频编辑器；classic 可用，main 正以 Rust core + plugin-first + MCP 从零重写（OpenCut-app, 2026）
- [[hyperframes]] — HeyGen 开源 HTML→确定性 MP4 框架：可 seek 动画 + Agent skills（19）+ CLI/Lambda/Catalog，面向编码 Agent 写视频（HeyGen, 2026）
- [[hypit]] — 面向编码 Agent 的开源视频生产框架：SVML 把爆款写成可重跑 workflow，词锚定字幕/B-roll/特效，一条命令出大量变体（hypit-ai, 2026）
- [[excalidraw]] — 开源手绘风格虚拟白板：无限画布、协作与 E2E 加密，开放 `.excalidraw` JSON 格式（excalidraw.com）
- [[excalidraw-diagram-generator]] — 从自然语言生成 `.excalidraw` 的 Agent Skill，覆盖九类图 + 模板/脚本（github/awesome-copilot）
- [[diagram-design]] — 编辑级自包含 HTML+SVG 图表 Skill：39 种类型、品牌 onboard、draw.io/Mermaid 重绘（cathrynlavery, 2026）
- [[design-agent]] — 团队级设计 Agent：Slack 线程 → 每线程沙箱（Chrome/FFmpeg/HyperFrames/D2）→ 品牌 token 约束 + 视觉自检 + 代码闸门 → 成品图/deck/图表/视频，Apache-2.0（LangChain, 2026）
