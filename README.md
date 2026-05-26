# Obsidian Vault

一个 **LLM-managed wiki**：原始素材丢进 `raw/`，由 AI Agent 增量编译成结构化、互相 wikilink 的 `wiki/` 文章，替代传统 RAG 做长期知识沉淀。

灵感来自 Karpathy 的 [llm-knowledge-bases](wiki/llm-knowledge-bases.md) 思路 —— 不再让 LLM 每次现搜原始资料，而是把它当成长期 wiki 编辑者，把知识"编译"成稳定、可链接、可追溯的文章。

## 目录结构

```
.
├── AGENTS.md          # 给 Agent 看的工作规则（最重要）
├── raw/               # 原始素材：网页剪藏、PDF 摘录、笔记。只增不改
├── wiki/              # LLM 编译生成的结构化文章
│   └── index.md       # 文章总索引（Agent 维护）
├── projects/          # 项目文件夹（overview.md / tasks.md / ideas.md）
├── diary/             # 日记，命名格式：YYYY-MM-DD [摘要].md
├── tasks.md           # 全局任务列表
├── .agents/           # 本仓库专属 Agent skills
├── .cursor/           # Cursor 规则与配置
└── .codegraph/        # CodeGraph 索引（结构化代码图谱缓存）
```

## 核心工作流

### 1. URL 剪藏

直接把一个或多个 URL 粘进对话，Agent 自动：

1. 抓取页面 → 转干净 Markdown
2. 存到 `raw/YYYY-MM-DD-short-title.md`，附 frontmatter（`url` / `clipped` / `title`）
3. 进入编译流程：在 `wiki/` 生成对应文章并更新 `index.md`

### 2. 编译

读取 `raw/` 中的素材，按以下规则产出 `wiki/` 文章：

- 文件名用英文短横线（如 `vue-reactivity-system.md`）
- frontmatter 含 `tags` / `created` / `source`
- 使用 `[[wikilink]]` 互联已有文章
- 结构：一句话摘要 → 核心概念 → 详细内容 → 相关链接
- 更新 `wiki/index.md`

### 3. 问答与研究

提问时，Agent 先搜 `wiki/` 已有文章综合作答；若答案有归档价值，会建议沉淀为新文章。

### 4. 健康检查

请求"健康检查"时，Agent 扫描 `wiki/` 找出：孤立文章、内容重叠可合并项、过时内容、`index.md` 与实际文件不一致之处。

## 当前内容速览

| 主题 | 文章 |
| --- | --- |
| LLM 知识管理 | [[llm-knowledge-bases]] |
| Agent 体系 | [[hermes-agent]] · [[agent-skills]] · [[impeccable]] · [[trellis]] |
| 工程方法 | [[openspec]] · [[codegraph]] |
| 设计 | [[open-design]] |

完整列表见 [`wiki/index.md`](wiki/index.md)。

## 红线

- `raw/` 是只读档案，**不修改、不删除**
- 不创建超过两层嵌套的目录
- 摘要不能丢失原文关键细节
- 不生成空壳占位文章

## 对 Agent 的入口

所有具体规则定义在 [`AGENTS.md`](AGENTS.md)，Cursor / Claude Code 等 Agent 进入仓库后应先读它。
