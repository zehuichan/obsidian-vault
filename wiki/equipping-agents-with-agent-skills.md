---
tags: [agent-skills, anthropic, progressive-disclosure, claude-code, security]
created: 2026-11-17
source: "raw/2026-11-17-anthropic-equipping-agents-with-agent-skills.md"
---

# Equipping Agents for the Real World with Agent Skills

Anthropic 对 Agent Skills 的官方介绍：Skill 是包含 `SKILL.md` 的目录，里面是指令、脚本和资源，Agent 可以动态发现并加载，把通用 Agent 变成专门化 Agent，就像给新员工写一份入职指南（Anthropic Engineering, 2025-10-16）。

> 本文是 Skill 概念的一手来源。[[agent-skills]] 基于第三方文章整理了规范结构、设计模式与评估方法，本文补充官方的设计动机、与代码执行的结合、官方开发建议、安全提示与路线图。

## 核心概念

- **动机**：Agent 越来越强，能与完整计算环境交互（如 Claude Code 用本地代码执行和文件系统），需要更**可组合、可扩展、可移植**的方式注入领域专长；不必再为每个场景定制碎片化 Agent
- **渐进式披露是核心设计原则**：像一本从目录、到章节、再到详细附录的手册，让 Claude 只在需要时加载信息
- **有文件系统和代码执行工具的 Agent，不需要把整个 Skill 读进上下文**，所以 Skill 能打包的上下文量实际上不受限

## Skill 的结构

以驱动 Claude 文档编辑能力的 PDF Skill 为例（Claude 本就懂 PDF，但难以直接操作，例如填表）：

- 最简形态是一个含 `SKILL.md` 的目录；文件必须以 YAML frontmatter 开头，必填 `name` 和 `description`
- **第一层**：启动时，Agent 把所有已安装 Skill 的 `name` 和 `description` 预载进 system prompt，让 Claude 知道何时该用哪个，又不必全部载入
- **第二层**：Claude 认为相关时读取完整 `SKILL.md` 正文
- **第三层及以后**：Skill 变复杂后，可在目录里捆绑更多文件，由 `SKILL.md` 按文件名引用；例如 PDF Skill 把填表说明放进 `forms.md`，只在填表时读，核心保持精简，另有 `reference.md`

**触发过程（以 PDF Skill 为例）**
1. 上下文里先有核心 system prompt、每个已装 Skill 的元数据和用户的首条消息
2. Claude 调用 Bash 工具读 `pdf/SKILL.md`，触发该 Skill
3. Claude 选择读取捆绑的 `forms.md`
4. 带着加载的指令去做用户任务

## Skill 与代码执行

- LLM 擅长很多事，但某些操作更适合传统代码：通过生成 token 排序一个列表，远比直接跑排序算法昂贵；许多应用还需要只有代码才有的确定性
- PDF Skill 带一个预写的 Python 脚本，读取 PDF 并抽取所有表单字段；Claude 可直接运行，**无需把脚本或 PDF 载入上下文**；代码是确定性的，所以流程一致、可重复
- 脚本可以按 Claude 的判断当作工具执行

## 开发与评估 Skill 的官方建议

- **从评估开始**：先让 Agent 在有代表性的任务上跑，观察它在哪里挣扎或需要额外上下文，再增量地构建 Skill 弥补缺口
- **为规模而结构化**：`SKILL.md` 太庞大就拆分并引用；互斥或很少同时使用的上下文，分开路径可降低 token 消耗；代码既可作可执行工具也可作文档，要明确 Claude 应该直接运行脚本还是读进上下文当参考
- **站在 Claude 的视角**：监控它在真实场景里如何使用你的 Skill，留意意外轨迹或对某些上下文的过度依赖；尤其注意 `name` 和 `description`，Claude 用它们决定是否触发
- **与 Claude 一起迭代**：做任务时让 Claude 把成功做法和常见错误沉淀为 Skill 里可复用的上下文和代码；跑偏时让它自我反思哪里出了问题，由此发现它真正需要什么上下文，而不是事先臆测

## 安全考量

- Skill 通过指令和代码赋予能力，恶意 Skill 可能在使用环境里引入漏洞，或诱导 Claude 外传数据、执行意外操作
- 只从可信来源安装；来源不够可信时先彻底审计：阅读捆绑的所有文件，尤其留意代码依赖和图片、脚本等捆绑资源；也要留意指示 Claude 连接不可信外部网络的指令或代码

## 现状与路线图

- 当时已在 Claude.ai、Claude Code、Claude Agent SDK 和 Claude Developer Platform 支持
- 计划持续补全创建、编辑、发现、分享、使用的全生命周期功能；探索 Skill 如何与 MCP 服务器互补，教 Agent 涉及外部工具和软件的复杂工作流
- 更远期：让 Agent 自己创建、编辑和评估 Skill，把自己的行为模式沉淀为可复用能力
- 后续动态：2026-10-22 Anthropic 把 Agent Skills 捐给 Agentic AI Foundation，已有 70 多个 Agent 平台集成（详见 [[anthropic-learning-path]]）

## 相关链接

- [[agent-skills]] — 规范结构、触发机制、设计模式与评估方法
- [[addy-osmani-agent-skills]] — 按该规范打包的整条 SDLC
- [[matt-pocock-skills]] — 可组合的工程 Skill 集
- [[impeccable]] — 完成度很高的 Skill 包范例
- [[effective-context-engineering]] — 渐进式披露与即时检索的上位框架
- [[anthropic-learning-path]] — Anthropic 官方学习路径总览

## 参考

- 原文: https://www.anthropic.com/research/equipping-agents-for-the-real-world-with-agent-skills
