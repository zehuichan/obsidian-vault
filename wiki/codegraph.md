---
tags: [ai-agent, mcp, code-intelligence, cursor, claude-code, developer-tools]
created: 2026-05-21
source: "raw/2026-05-21-codegraph.md"
---

# CodeGraph

CodeGraph 是为 AI 编码 Agent 构建的**本地预索引代码知识图谱**，通过 MCP 暴露符号搜索、调用链分析和上下文构建能力，让 Agent 用少量工具调用替代 grep/glob/Read 的盲目扫描，在 7 个真实开源项目基准测试中平均节省 35% 成本、59% token、70% 工具调用。

## 核心概念

- **预索引 vs 实时扫描**：Agent 探索代码库时通常派生子 Agent 用 grep、glob、Read 逐文件搜索；CodeGraph 事先用 tree-sitter 解析 AST，把符号、调用关系、导入、继承等存入本地 SQLite 图数据库
- **MCP 集成**：以 MCP Server 形式接入 Claude Code、Cursor、Codex CLI、opencode，Agent 通过 `codegraph_search`、`codegraph_context` 等工具直接查询图谱
- **100% 本地**：数据不离开机器，无需 API Key，索引存储在 `.codegraph/codegraph.db`
- **自动同步**：MCP Server 监听 OS 原生文件事件（FSEvents/inotify/ReadDirectoryChangesW），2 秒 debounce 后增量更新索引
- **框架感知路由**：识别 13 种 Web 框架的路由定义（Django、FastAPI、Express、NestJS、Spring 等），将 URL 模式链接到 handler 符号

## 架构与工作流程

```
Agent 提问 → MCP Server 接收
    → Search / Callers / Context 工具查询
    → SQLite 图数据库（符号 + 边 + FTS5 全文索引）
    → 返回相关符号、调用链、源码片段
```

四个阶段：

1. **Extraction** — tree-sitter 解析源码，语言特定 query 提取节点（函数、类、方法）和边（调用、导入、继承）
2. **Storage** — 写入 `.codegraph/codegraph.db`，FTS5 支持按名称全文搜索
3. **Resolution** — 解析函数调用→定义、import→源文件、类继承、框架路由模式
4. **Auto-Sync** — 文件变更时 debounce 增量同步，图谱随编码实时更新

## 基准测试亮点

在 7 个跨语言真实代码库上，Claude Code（Opus 4.7）回答一个架构问题时，启用 CodeGraph 对比未启用的中位数结果：

| 指标 | 平均节省 |
|------|---------|
| 成本 | 35% |
| Token | 59% |
| 时间 | 49% |
| 工具调用 | 70% |

收益随代码库规模放大：VS Code（~10k 文件）从 23 次工具调用降至 7 次、token 从 1.4M 降至 393k；小仓库如 Gin（~150 文件）差距较小。关键原因是启用索引后 Agent 通常只需 `codegraph_context` + 一次 `codegraph_explore` 即可回答，零文件读取。

## MCP 工具一览

| 工具 | 用途 |
|------|------|
| `codegraph_search` | 按名称搜索符号 |
| `codegraph_context` | 为任务构建相关代码上下文 |
| `codegraph_callers` / `codegraph_callees` | 追踪调用链 |
| `codegraph_impact` | 变更前分析影响范围 |
| `codegraph_node` | 获取单个符号详情（可选含源码） |
| `codegraph_files` | 获取索引文件结构 |
| `codegraph_status` | 检查索引健康状态 |

**Agent 使用策略**：主会话应避免直接调用 `codegraph_explore`/`codegraph_context`（返回大量源码会填满上下文），应派生 Explore 子 Agent 使用这些工具；主会话仅使用轻量工具做定向查找。

## 快速上手

```bash
# 1. 交互式安装（自动检测并配置 Agent）
npx @colbymchenry/codegraph

# 2. 重启 Agent 使 MCP 生效

# 3. 在项目中初始化索引
cd your-project
codegraph init -i
```

非交互安装：

```bash
codegraph install --target=cursor,claude --yes   # 指定 Agent
codegraph install --yes                          # 自动检测
```

Cursor 用户会获得 `.cursor/rules/codegraph.mdc` 指令文件；Claude Code 写入 `~/.claude.json` MCP 配置和 `CLAUDE.md` 使用指南。

## 常用 CLI

| 命令 | 说明 |
|------|------|
| `codegraph init -i` | 初始化项目并建立索引 |
| `codegraph index` | 全量索引（`--force` 重建） |
| `codegraph sync` | 增量更新 |
| `codegraph status` | 查看统计和后端类型 |
| `codegraph query <name>` | 搜索符号 |
| `codegraph context <task>` | 构建 AI 上下文 |
| `codegraph affected [files...]` | 追踪变更影响的测试文件（可接 `git diff --stdin`） |
| `codegraph serve --mcp` | 启动 MCP Server |

## 配置要点

`.codegraph/config.json` 控制索引行为：

- `languages` — 索引语言（空则自动检测）
- `exclude` — 忽略 glob（默认含 `node_modules/**`）
- `maxFileSize` — 跳过大文件（默认 1MB）
- `extractDocstrings` / `trackCallSites` — 提取文档字符串和调用位置

支持 19+ 语言，含 TypeScript、Python、Go、Rust、Java、C#、Swift、Vue、Svelte、Pascal/Delphi 等。

## 与其他方案的关系

CodeGraph 解决的是**代码库内探索效率**问题，与 [[llm-knowledge-bases]] 管理的**跨文档/wiki 知识复利**互补：前者索引源码符号图，后者编译外部素材为结构化 wiki。两者都可作为 AI Agent 的持久知识层，但数据源和查询模式不同。

与 [[agent-skills]] 的关系：CodeGraph 通过 MCP + 指令文件（如 `.cursor/rules/codegraph.mdc`）引导 Agent 行为，类似 Skill 的触发与执行约束，但更侧重结构化代码索引而非通用任务流程。

## 注意事项

- **必须初始化**：项目需存在 `.codegraph/` 目录，Agent 才会启用 CodeGraph 工具
- **后端性能**：优先使用 native `better-sqlite3`；WASM fallback 慢 5-10x 且可能导致 `database is locked`
- **直接查询才有效**：若 Agent 仍派生子 Agent 用 grep/Read 探索，CodeGraph 反而成为开销；指令文件专门约束这一行为
- **小仓库收益有限**：~150 文件以下项目原生搜索已足够快

## 相关链接

- [[agent-skills]] — Agent Skill 规范与渐进式加载
- [[llm-knowledge-bases]] — LLM 管理的持久 wiki 知识库模式
- [[hermes-agent]] — 具备 skill 自动创建能力的开源 Agent
- [[openspec]] — spec-driven 规划层框架
