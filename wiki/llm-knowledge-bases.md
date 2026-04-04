---
tags: [llm, knowledge-management, obsidian, workflow, rag]
created: 2026-04-04
source: "https://x.com/karpathy/status/2039805659525644595"
---

# LLM Knowledge Bases

利用 LLM 将零散的原始资料（文章、论文、代码库等）增量编译为结构化的 Markdown wiki，再以 Obsidian 作为前端进行浏览、查询和可视化的个人知识管理方法。由 Andrej Karpathy 于 2026 年 4 月提出。

## 核心概念

- **LLM 作为编辑者**：人类不直接编写 wiki，而是让 LLM 全权负责文章的创建、分类、摘要和交叉链接
- **编译式知识管理**：原始素材（raw/）经 LLM "编译"为结构化的 wiki 文章，类似源码到产物的关系
- **增量维护**：每次新增素材后，LLM 增量更新 wiki，而非全量重建
- **知识复利**：查询的输出可以回流到 wiki 中，使探索过程本身不断丰富知识库

## 工作流

### 1. 数据收集（Data Ingest）

将各类来源的文档索引到 `raw/` 目录：

- 使用 Obsidian Web Clipper 浏览器扩展将网页转为 Markdown
- 配合快捷键下载关联图片到本地，方便 LLM 引用
- 素材类型涵盖文章、论文、代码仓库、数据集、图片等

### 2. 编译（Compile）

LLM 读取 `raw/` 中的原始素材，执行：

- 生成摘要和分类
- 提炼概念，为每个概念写独立文章
- 建立文章间的反向链接（backlinks）
- 维护索引文件

### 3. 浏览与查询（IDE + Q&A）

以 Obsidian 作为"前端 IDE"，支持：

- 浏览原始数据和编译后的 wiki
- 向 LLM 提出复杂问题，由 LLM 在 wiki 中检索并综合回答
- 在约 100 篇文章、40 万词规模下，LLM 通过索引文件和摘要即可有效检索，无需复杂的 RAG 管线

### 4. 输出与回流（Output）

LLM 的回答不局限于纯文本，可以生成：

- Markdown 文件
- 幻灯片（Marp 格式）
- 图表（matplotlib）
- 其他可视化格式

输出结果可回流归档到 wiki，形成**知识复利**循环。

### 5. 健康检查（Linting）

定期对 wiki 执行 LLM "lint"：

- 发现数据不一致之处
- 通过联网搜索补全缺失数据
- 挖掘文章间的潜在关联，建议新文章候选
- 提升整体数据完整性

### 6. 扩展工具（Extra Tools）

可根据需要开发辅助工具，例如：

- 基于 wiki 的简易搜索引擎（Web UI + CLI）
- CLI 工具供 LLM 在大规模查询时调用

## 进阶方向

- **合成数据 + 微调**：当知识库足够大时，可以考虑用 wiki 内容生成合成训练数据，微调 LLM 使其将知识内化到权重中，而非仅依赖上下文窗口

## 关键洞察

> "You rarely ever write or edit the wiki manually, it's the domain of the LLM."
> — Andrej Karpathy

这一方法的本质是将 LLM 从"代码操作者"转变为"知识操作者"——token 的消耗从编写代码转向编织知识。Karpathy 认为这里存在一个巨大的产品机会，而非仅仅是一堆拼凑的脚本。

## 相关链接

- 原文：[Andrej Karpathy on X](https://x.com/karpathy/status/2039805659525644595)
