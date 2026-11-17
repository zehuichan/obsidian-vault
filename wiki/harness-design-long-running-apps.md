---
tags: [harness, multi-agent, evaluator, long-running-agents, anthropic, frontend-design]
created: 2026-11-17
source: "raw/2026-11-17-anthropic-harness-design-long-running-apps.md"
---

# Harness Design for Long-Running Application Development

借鉴 GAN 的「生成器 + 评估器」结构，再加上 planner，构成三 Agent harness，能在数小时自主编码里产出完整全栈应用；并且随模型升级，应逐个拆除不再承重的组件（Anthropic，2026-03-24）。

## 核心概念

- **朴素实现的两个问题**：① 长任务中上下文填满会失去连贯，部分模型出现「context anxiety」（接近自认的上限时过早收尾）；② 自我评估失真——Agent 评价自己产出时倾向自信地夸赞，主观任务（如设计）尤其明显
- **context reset vs compaction**：compaction 在原地总结历史，保持连续但不给干净起点，context anxiety 仍可能存在；reset 清空窗口并启动新 Agent，靠结构化交接物传递状态，代价是编排复杂度、token 开销和延迟
- **生成与评估分离**：评估器仍是 LLM，也偏宽容，但调教一个独立的「怀疑型」评估器，比让生成器自我批判容易得多；评估器的反馈给生成器提供了具体可迭代的对象
- **harness 的每个组件都编码了「模型自己做不到什么」的假设**，这些假设需要压测，也会随模型升级过时

## 前端设计：把主观质量变成可评分

四条评分标准同时放进生成器和评估器的 prompt：

- **Design quality**：配色、字体、布局、图像等是否形成统一的情绪和身份
- **Originality**：有无自定义决策，还是模板、库默认值和 AI 套路（如白卡片上的紫色渐变）
- **Craft**：字体层级、间距、色彩和谐、对比度等技术功底
- **Functionality**：不依赖美学的可用性

作者更看重 design quality 和 originality，因为 Claude 默认在 craft 与 functionality 上已不错，但设计上常是平庸的。评估器用带详细打分拆解的 few-shot 示例校准，减少迭代间的分数漂移。

流程：生成器基于 Claude Agent SDK 写 HTML/CSS/JS；评估器通过 Playwright MCP 实际操作页面、截图研究，再逐条打分并写评语；每次生成跑 5–15 轮，最长约 4 小时；生成器每轮要做战略决策：分数向好则优化当前方向，否则彻底转向另一种审美。

观察：
- 评分标准的措辞会引导输出，如「最好的设计是博物馆级」会让设计趋向某种视觉收敛；即便第一轮，也明显好于无 prompt 基线
- 分数整体上升但不总是线性，有时中间轮次比最后一轮更好；实现复杂度随轮次增加
- 荷兰美术馆网站例子：第 9 轮是干净的深色落地页，第 10 轮推翻重做，变成用 CSS 透视渲染的 3D 展厅，以门廊导航

## 全栈：三 Agent 架构

- **Planner**：把 1–4 句 prompt 扩写成完整产品 spec；要有野心地定范围，但只写产品背景和高层技术设计，避免过早写细节、一旦错误就连锁到下游；还被要求把 AI 功能织入产品
- **Generator**：按 sprint 一次做一个功能，技术栈 React + Vite + FastAPI + SQLite（后改 PostgreSQL），sprint 结束先自评，再交 QA，用 git 管版本
- **Evaluator**：用 Playwright MCP 像用户一样点击运行中的应用，测试 UI、API 端点和数据库状态；按产品深度、功能、视觉设计、代码质量打分，每项有硬阈值，任何一项不过则该 sprint 失败，并给生成器详细反馈
- **Sprint contract**：每个 sprint 开工前，生成器与评估器先协商「完成」的定义和验证方式，直到达成一致，再动手写代码；弥合高层 spec 与可测实现之间的缺口
- **通信**：通过文件，一个 Agent 写文件，另一个读取并在文件内或用新文件回应
- Opus 4.5 基本消除了 context anxiety，因此该 harness 去掉了 context reset，整个构建作为一个连续会话，靠 SDK 的自动 compaction

## 实验结果

**v1 harness（Opus 4.5）**，提示词：做一个 2D 复古游戏制作器（关卡编辑器、精灵编辑器、实体行为、可玩测试模式）

| Harness | 时长 | 成本 |
| --- | --- | --- |
| Solo | 20 分钟 | $9 |
| 完整 harness | 6 小时 | $200 |

- 成本超过 20 倍，但质量差异一目了然：solo 版实体能显示却对输入无响应，实体定义与游戏运行时的接线是坏的；完整版 planner 扩成 16 个功能、10 个 sprint，含动画系统、行为模板、音效音乐、AI 辅助精灵和关卡生成、导出分享等，play mode 真能玩
- 评估器的发现具体到文件和行号，如矩形填充工具只在拖拽起止点放 tile、Delete 键处理需要 `selection` 与 `selectedEntityId` 同时存在、FastAPI 路由顺序导致 `/frames/reorder` 被当成 `frame_id` 返回 422；仅 Sprint 3 就有 27 条关卡编辑器验收标准
- **QA 需要调教**：开箱即用的 Claude 是糟糕的 QA，会发现问题后又说服自己「不严重」而放行，且只做浅层测试。调法是读评估器日志，找出与自己判断不一致的地方，改 QA 的 prompt，反复数轮

**简化 harness（Opus 4.6）**
- 最初大刀阔斧精简，无法复现原有表现，也看不出哪些部分是承重的；于是改为一次去掉一个组件，观察对结果的影响；同时 Opus 4.6 发布，进一步促使降低复杂度
- 去掉 sprint 构造，模型已能原生处理；保留 planner（没有它，生成器会 under-scope，做出功能更少的应用）；评估器改为在运行结束时单次评估
- **评估器不是固定的是/否选择**：当任务超出当前模型单独可靠完成的范围时它才值得；落在范围内时就是额外开销。Opus 4.5 时评估器在整个构建里都抓到问题，4.6 上边界外移，只有仍处于生成器能力边缘的部分才有明显收益
- 还加强了提示，让生成器构建真正能通过工具驱动应用功能的内置 Agent

**DAW 运行**（提示词：在浏览器里用 Web Audio API 做一个功能完整的 DAW）：总耗时 3 小时 50 分，成本 $124.70

| 阶段 | 时长 | 成本 |
| --- | --- | --- |
| Planner | 4.7 分钟 | $0.46 |
| Build 第 1 轮 | 2 小时 7 分 | $71.08 |
| QA 第 1 轮 | 8.8 分钟 | $3.24 |
| Build 第 2 轮 | 1 小时 2 分 | $36.89 |
| QA 第 2 轮 | 6.8 分钟 | $3.09 |
| Build 第 3 轮 | 10.9 分钟 | $5.88 |
| QA 第 3 轮 | 9.6 分钟 | $4.06 |
| 合计 | 3 小时 50 分 | $124.70 |

QA 仍抓到真实缺口：片段不能在时间线拖动、没有乐器 UI、没有图形化效果器、录音只是占位、片段边缘拖拽调整和切分未实现。局限：Claude 听不到声音，音乐品味方面的 QA 反馈偏弱。

## 经验总结

- 针对所用模型做实验，读它在真实问题上的 trace，并据此调优
- 复杂任务有时能通过拆解、给每个子问题配专门 Agent 获得提升
- 新模型发布后，重新审视 harness：剥掉不再承重的部分，加上能实现之前做不到事情的新部件
- 作者的信念：有趣的 harness 组合空间不会随模型变强而缩小，而是会移动

## 相关链接

- [[effective-harnesses-long-running-agents]] — 本文的前身：initializer + coding agent
- [[effective-context-engineering]] — compaction 与 reset 的背景
- [[demystifying-agent-evals]] — 评估器调教与 grader 设计的通用方法
- [[impeccable]] — 同样把设计原则编码成可评审标准的 Skill 包
- [[design-agent]] — 用 brand-lint 等闸门和视觉自检约束输出，与评估器思路相通
- [[agent-browser]] — Playwright 之外的浏览器自动化选择，可作评估器的操作工具
- [[anthropic-learning-path]] — Anthropic 官方学习路径总览

## 参考

- 原文: https://www.anthropic.com/research/harness-design-long-running-apps
