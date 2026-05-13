---
tags: [ai-agent, agent-skills, prompt-engineering, workflow, tool-use]
created: 2026-05-12
source: "raw/2026-05-12-agent-skill-spec-patterns.md"
---

# Agent Skills

Agent Skill 是一种可复用的 Prompt 增强包，把任务触发条件、领域知识、操作流程、工具脚本和输出边界封装成可按需加载的结构化行为单元，而不是把所有指令一次性塞进 system prompt。

## 核心概念

- **Skill 不是普通 Prompt**：Prompt 更像一次性指令，Skill 则是围绕可重复任务设计的行为模块，包含触发描述、执行步骤、参考资料、脚本和资产
- **渐进式加载**：会话启动时只加载 `name` 和 `description`；任务匹配后才加载 `SKILL.md` 正文；正文引用的 `scripts/`、`references/`、`assets/` 再按需读取
- **触发依赖 description**：模型根据 `description` 自主判断是否启用 Skill，因此 description 应描述用户意图和适用场景，而不是复述内部工作流
- **Skill 开发需要评估闭环**：高质量 Skill 应通过测试场景、A/B 对比、评分、人工反馈和迭代优化来避免凭感觉堆提示词
- **设计模式决定内部结构**：同样是 `SKILL.md`，可以是专家知识包装、模板生成器、审查器、需求访谈器或多阶段流水线

## 规范结构

一个 Skill 的最小形态是一个目录加 `SKILL.md`：

```text
skill-name/
├── SKILL.md
├── scripts/
├── references/
└── assets/
```

`SKILL.md` 由 YAML frontmatter 和 Markdown 正文组成。`name` 与 `description` 是核心字段：`name` 应与目录名一致，使用小写字母、数字和连字符；`description` 则承担触发入口的职责，要清楚说明这个 Skill 做什么、什么时候使用，并包含模型识别任务所需的关键词。

`scripts/` 放可执行工具，适合把重复出现的辅助逻辑固化下来；`references/` 放按需读取的技术参考、检查清单或领域规范；`assets/` 放模板、Schema、图片、示例数据等静态资源。关键原则是让 `SKILL.md` 保持精简，把大块资料拆到可按需加载的文件中。

## 渐进式加载

Agent Skills 的核心机制是三层加载：

- **L1 目录层**：启动时加载所有 Skill 的 `name + description`，让模型知道有哪些能力可用，单个 Skill 通常只消耗几十到一百 tokens
- **L2 指令层**：当用户任务匹配 description 时，读取完整 `SKILL.md` 正文，获得工作流、边界条件和输出格式
- **L3 资源层**：只有在正文明确要求时，才读取 `scripts/`、`references/` 或 `assets/` 中的文件

这种设计用少量上下文暴露大量潜在能力，适合 [[hermes-agent]] 这类需要跨任务复用技能和记忆的 Agent 系统，也呼应 [[llm-knowledge-bases]] 中“把可复用知识编译为持久结构”的思想。

## 构建方法

Anthropic 的 Skill-Creator 把 Skill 开发当作类似机器学习的工程流程：先捕获需求，再编写 Skill，设计测试用例，并用 with-skill / without-skill 两组子 Agent 做 A/B 测试。之后通过 Grader、Comparator、Analyzer 分别完成断言评分、盲比较和模式分析，最后根据人工反馈迭代。

这个方法的价值在于把 Prompt Engineering 从“凭感觉改指令”推进到“有训练集、测试集、评估指标和防过拟合机制”的工程过程。文章特别强调：遇到失败时不应只叠加更多 `ALWAYS` / `NEVER`，而要解释为什么某个行为重要，并抽取跨测试用例重复出现的模式。

Superpowers 的 Writing-Skills 则更强调 TDD：先在没有 Skill 的情况下运行压力场景，观察 Agent 会如何违规或合理化；再写最小 Skill 让它通过；最后针对新出现的漏洞迭代修正。这种方法尤其适合纪律执行型 Skill，例如测试先行、系统化调试、完成前验证等。

## 设计模式

文章归纳了五类常见 Skill 设计模式：

- **Tool Wrapper**：封装某个框架、库、团队规范或领域知识，让 Agent 在需要时读取参考资料并按规则执行
- **Generator**：通过模板和风格指南生成一致的结构化输出，适合报告、API 文档、脚手架等
- **Reviewer**：把检查清单与审查流程分离，Agent 按清单逐项检查并解释问题原因，适合 PR 审查、安全扫描和质量评估
- **Inversion**：让 Agent 先访谈用户，收集完整需求后再行动，适合项目规划、架构设计和需求不明确的任务
- **Pipeline**：把复杂任务拆成严格顺序步骤，每步都有输入、输出和检查点，适合多阶段文档生成、代码生成或人工确认流程

这些模式可以组合使用，例如 Pipeline + Reviewer 在流水线末尾加入自动质量检查，Generator + Inversion 先补齐用户输入再填充模板，Inversion + Pipeline 先完成需求澄清再进入执行阶段。

## 风险与局限

Skill-Creator 的完整流程很严谨，但也有明显成本：评估和 description 优化可能启动大量子 Agent，token 消耗高且不透明；完整流程需要多次用户确认，对简单 Skill 可能过重；多轮迭代容易让 Skill 膨胀，违背精简和按需加载的初衷。

另一个关键限制是触发机制并不总由 description 质量决定。某些“操作型 Skill”如果模型认为自己可以直接完成任务，可能不会主动读取 Skill；这类 Skill 需要重新考虑触发场景、任务复杂度和是否真的需要封装。

## 实践原则

- `description` 只写触发条件，不要总结完整工作流，否则 Agent 可能直接按 description 执行而跳过正文
- 正文默认保持短小，只补充模型不知道的知识、项目专属约束和必须遵守的流程
- 对脆弱或一致性要求高的任务降低自由度，对开放性任务保留判断空间
- 重复出现的辅助脚本应移入 `scripts/`，大型参考资料应移入 `references/`
- 对纪律执行型 Skill 使用压力场景测试，对参考资料型 Skill 重点测试信息是否可发现、是否能正确应用

## 相关链接

- [[hermes-agent]] — 自我改进 Agent 中的 Skills 系统实践
- [[llm-knowledge-bases]] — 将知识从临时上下文编译进持久结构的方法论
- [[impeccable]] — 一个完成度极高的 Skill 包范例：单命名空间、23 子命令、按 register 分支的引用资料、可 pin/unpin 重定向、A/B 评估闭环
- [[open-design]] — Skill-driven 设计工作流的另一条路径（31 skills + 72 design systems）

## 参考

- 原文: https://mp.weixin.qq.com/s/LCpiLyLnRn5WyuHpribyHw
- Agent Skills 开放规范: https://agentskills.io/specification
- Anthropic Skills 仓库: https://github.com/anthropics/skills
- Superpowers 框架: https://github.com/obra/superpowers
- Awesome Agent Skills: https://github.com/VoltAgent/awesome-agent-skills
