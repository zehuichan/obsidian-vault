---
tags: [context-engineering, ai-agent, anthropic, long-horizon, prompt-engineering]
created: 2026-11-17
source: "raw/2026-11-17-anthropic-effective-context-engineering.md"
---

# Effective Context Engineering for AI Agents

Context engineering 是 prompt engineering 的自然延伸：不再只琢磨怎么写 prompt，而是在推理的每一步，策划并维护进入模型有限注意力预算的那组最小高信号 token（Anthropic Applied AI 团队，2025-09-29）。

## 核心概念

- **Context 是有限资源**：随 token 增多，模型回忆信息的准确度下降（context rot）；根源是 transformer 中每个 token 要和所有 token 建立 n² 个两两关系，且训练数据里短序列更常见。性能是渐变梯度而不是悬崖
- **指导原则**：找到「最小的高信号 token 集合」，最大化期望结果出现的概率；最小不等于短，仍需给足信息
- **Prompt 工程 vs Context 工程**：前者是写好指令（尤其 system prompt）的离散任务；后者是循环迭代的策划过程，覆盖 system 指令、工具、MCP、外部数据、消息历史等全部状态，每次决定传给模型什么时都要做一次
- **Agent 的定义**：LLM 在循环里自主使用工具
- **长时任务的三类手段**：compaction（压缩）、structured note-taking（结构化笔记 / agentic memory）、sub-agent 架构

## 有效 Context 的构成

**System prompt：找对「高度」**
- 两个失败极端：把复杂脆弱的 if-else 逻辑硬编码进 prompt（脆弱、难维护）；或过于笼统、默认模型与你共享上下文
- 最佳高度：足够具体以引导行为，又留给模型强启发式的弹性
- 建议用 XML 标签或 Markdown 标题分区（背景信息、工具指引、输出描述等），但随模型变强，具体格式越来越不重要
- 做法：先用最强模型测试一个最小 prompt，再根据失败模式补充指令和示例

**工具**
- 工具定义了 Agent 与信息/行动空间的契约，返回的信息要省 token，也要鼓励高效行为
- 工具应自包含、对错误鲁棒、用途极清晰，参数描述明确无歧义
- 最常见的失败：工具集臃肿、功能重叠，导致「该用哪个」含糊。如果人类工程师都说不清该用哪个，Agent 也不会更好
- 精简的最小可用工具集也更利于长交互中维护和裁剪上下文

**示例（few-shot）**
- 不要把一长串边界情况塞进 prompt；应策划一组多样、典型的示例来体现期望行为，示例是「胜过千言万语的图片」

## 上下文检索与 Agentic Search

- **预先检索（embedding 检索）→ 即时检索（just in time）**：Agent 只保存轻量标识符（文件路径、已存查询、网页链接），在运行时通过工具动态加载数据。Claude Code 用它对大型数据库做分析：写针对性查询、存结果、用 `head` / `tail` 等命令，而不把整份数据读入上下文
- **元数据本身就是信号**：`tests` 目录里的 `test_utils.py` 与 `src/core_logic/` 里的同名文件意义不同；目录层级、命名、时间戳都帮助 Agent 判断如何使用信息
- **渐进式披露**：Agent 通过探索逐层建立理解，只把必要信息留在工作记忆，用笔记补充持久化
- **代价**：运行时探索比预计算检索慢，且需要精心设计工具和启发式，否则 Agent 会误用工具、钻死胡同或漏掉关键信息
- **混合策略**：Claude Code 即如此——`CLAUDE.md` 直接预载进上下文，glob / grep 等原语用于即时检索，绕开过期索引和复杂语法树问题。变化较少的领域（法律、金融）可能更适合混合策略
- 总体建议：「做最简单且有效的事」；随模型变强，会趋向让模型更自主、减少人工策划

## 长时任务的上下文工程

等更大的上下文窗口并不能解决问题：各种尺寸的窗口在可预见的将来都会有上下文污染和相关性问题。三类技术：

**Compaction（压缩）**
- 把接近窗口上限的对话总结后，用摘要开启新窗口，是提升长期连贯性的第一杠杆
- Claude Code 的做法：让模型总结并压缩关键细节，保留架构决策、未解决的 bug 和实现细节，丢弃冗余的工具输出；之后带着压缩后的上下文加最近访问的 5 个文件继续
- 艺术在于取舍：过度压缩会丢掉当时看不出、之后才重要的细节。调优时先最大化召回，再迭代提升精确度
- 最安全、最轻量的形式是 **tool result clearing**：深层历史里的原始工具结果不再需要，Claude Developer Platform 已推出该功能

**结构化笔记（Agentic memory）**
- Agent 定期把笔记写到上下文之外的持久存储，稍后再读回，如 Claude Code 的 to-do 列表或自定义的 `NOTES.md`
- 例子：Claude 玩 Pokémon，能跨数千步精确统计、自行绘制已探索区域的地图、记录战斗策略，上下文重置后读笔记接着训练或探索
- Sonnet 4.5 发布时，Developer Platform 推出了基于文件系统的 memory tool 公测版

**Sub-agent 架构**
- 专门化的子 Agent 用干净窗口处理聚焦任务，可能用掉数万 token 探索，但只返回 1,000–2,000 token 的精炼摘要；主 Agent 负责高层规划和综合
- 在复杂研究任务上相对单 Agent 有显著提升（见 Anthropic 多 Agent 研究系统一文）

**如何选择**
- 需大量往返对话的任务 → compaction
- 有明确里程碑的迭代式开发 → 笔记
- 可并行探索的复杂研究分析 → 多 Agent

## 与本库的关系

- [[agent-skills]] 的三层渐进式加载是这里「渐进式披露」和「即时检索」的具体落地
- [[headroom]] 做的是工具输出、日志、历史的上下文压缩，对应本文的 compaction / tool result clearing
- [[codegraph]] 用预索引代码图谱代替盲目 grep，是「预先检索」与「即时检索」之间的一种折中
- [[llm-knowledge-bases]] 把知识增量编译进持久 wiki，对应结构化笔记的进阶形态，详见 [[agent-memory-components]]

## 相关链接

- [[effective-harnesses-long-running-agents]] — 用 progress 文件 + git 在多窗口间交接
- [[harness-design-long-running-apps]] — context reset 与 compaction 的取舍、context anxiety
- [[agent-memory-components]] — 把笔记做成记忆体系的完整论述
- [[anthropic-learning-path]] — Anthropic 官方学习路径总览

## 参考

- 原文: https://www.anthropic.com/research/effective-context-engineering-for-ai-agents
