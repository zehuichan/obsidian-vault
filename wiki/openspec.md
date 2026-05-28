---
tags: [llm, agent, spec-driven, planning, open-source, cursor, brownfield, artifact-guided]
created: 2026-05-20
updated: 2026-05-28
source: "https://openspec.dev/"
---

# OpenSpec

由 Fission AI 维护的**轻量级 spec-driven 开发（SDD）框架**：在编码前，让人与 AI 先就「要做什么」达成一致——以 Markdown 规格（spec）和「变更包（change）」的形式落进仓库。开源、无需 API Key、不依赖 MCP；通过 `npm install -g @fission-ai/openspec@latest` 安装，对 Cursor、Claude Code、Codex、Copilot 等 25+ 编码 Agent 提供原生 slash 命令与 skill 集成。要求 Node.js ≥ 20.19.0。截至 2026-05，GitHub 51.3k stars，最新版 v1.3.1（2026-04-21）。

设计哲学（官方原话）：

> fluid not rigid · iterative not waterfall · easy not complex · built for brownfield not just greenfield · scalable from personal projects to enterprises

## v1.3 关键变化：artifact-guided 工作流

OpenSpec 在 v1.3 重写了核心工作流，从一个大命令 `/openspec:proposal` 改为**三段式 artifact-guided** 流程，所有命令统一前缀 `/opsx:`：

```
/opsx:propose <idea>   → 生成 openspec/changes/<id>/
                          ├── proposal.md   为什么做、要改什么
                          ├── specs/        新需求 & 验收场景
                          ├── design.md     技术方案
                          └── tasks.md      实施清单
/opsx:apply            → AI 按 tasks.md 逐项实现，并打勾
/opsx:archive          → 归档到 openspec/changes/archive/<date>-<id>/
                          同时把变更合并进 openspec/specs/
```

「artifact-guided」的含义：每一阶段都对应仓库里**具体可审查的文件**（artifact），而不是聊天历史里的某段话。审查者先看 proposal / spec delta / tasks，再决定是否进入实现。

## 核心概念

- **Spec 即活文档**：按能力（capability）组织在 `openspec/specs/<capability>/spec.md`，与代码同仓、可 git 协作
- **Change 包**：每次提案在 `openspec/changes/<id>/` 下生成完整 artifact 套件（proposal / specs / design / tasks），审查通过后归档
- **Spec delta**：变更以「需求增删」形式展示（SHALL 表述 + GIVEN/WHEN/THEN Scenario），审查「意图」而非逐行代码
- **Brownfield-first**：面向已有代码库，按需增量建 spec，反对一次性全量生成
- **Agent 无关**：规格不绑定某一 LLM 产品，换 Agent 仍可沿用同一套规划层

## 目录结构

```
openspec/specs/                  # 按能力划分的现行规格（活文档）
├── auth-session/spec.md
└── checkout-payment/spec.md

openspec/changes/<id>/           # 进行中的变更审查包
├── proposal.md
├── design.md
├── tasks.md
└── specs/                       # 该变更影响的 spec delta

openspec/changes/archive/        # 已归档的历史变更
└── 2026-01-23-add-dark-mode/
```

## Profile 系统：default vs expanded

OpenSpec 通过 **profile** 控制项目里激活哪一组 `/opsx:` 命令：

| Profile     | 暴露的命令                                                                                                | 适用场景                          |
| ----------- | -------------------------------------------------------------------------------------------------- | ----------------------------- |
| `default`   | `/opsx:propose` · `/opsx:apply` · `/opsx:archive`                                                  | 个人项目、轻量协作、想最快跑通三段式            |
| `expanded`  | 上述 + `/opsx:new` · `/opsx:continue` · `/opsx:ff` · `/opsx:verify` · `/opsx:bulk-archive` · `/opsx:onboard` | 团队项目、长周期 change、多人接力、批量归档     |

切换：`openspec config profile` 选择 profile，再用 `openspec update` 刷新 AI 指令到 `.cursor/rules/`、`.claude/`、`CLAUDE.md`、`AGENTS.md` 等位置。

## OpenSpec Dashboard

v1.3 引入的可视化看板，用于浏览仓库内所有 change、查看 spec 现状、跟踪 task 进度，与命令行 / Agent 操作互补。适合在 PR review 前快速看「这次变更碰了哪些 capability、还有几个任务未打勾」。

## 多语言与社区 schemas

- **Multi-Language**：spec 与 proposal 可用非英语撰写，工作流提示也支持本地化
- **Community schemas**：第三方维护的 schema bundle，作为独立仓库分发，把 OpenSpec 与具体工具链（CI、issue tracker、设计系统等）做意见性集成；类似 `github/spec-kit` 的社区扩展目录。可在 customization 文档里浏览 catalog

## 与同类方案的关系

来自 README 的官方对比，加上本知识库的延伸：

| 方案                     | 定位                                  | OpenSpec 的差异                                  |
| ---------------------- | ----------------------------------- | --------------------------------------------- |
| **Spec Kit**（GitHub）   | 重量级 SDD，强阶段门（phase gate）、大量 Markdown、依赖 Python | OpenSpec 更轻、允许自由迭代、纯 Node CLI                  |
| **Kiro**（AWS）          | 强大但绑定自家 IDE 与 Claude 系列模型           | OpenSpec 与现有工具链共存，不锁定 Agent                    |
| **裸 AI 编码**             | 无 spec 层，靠聊天历史                       | OpenSpec 用最少仪式带来可预测性                           |
| **[[trellis]]**        | 团队级 harness：spec + task + workflow + journal + hook | OpenSpec 是其中 spec/change 子集的轻量先行者；Trellis 把工作流变成 hook + skill 强约束 |
| **Agent 内置 Plan Mode** | 单次对话内规划                              | OpenSpec 把规划产物落仓、可 PR 共享、跨会话留存                  |

也就是说：

- **个人项目 / 想要最少仪式** → OpenSpec default profile
- **团队多人接力 / 需要平台 hook 自动注入** → [[trellis]]
- **只在某次对话里规划** → Agent 自带 Plan Mode 已够

## 工作流示例

```
You: /opsx:propose add-dark-mode
AI:  Created openspec/changes/add-dark-mode/
     ✓ proposal.md — why we're doing this, what's changing
     ✓ specs/       — requirements and scenarios
     ✓ design.md    — technical approach
     ✓ tasks.md     — implementation checklist
     Ready for implementation!

You: /opsx:apply
AI:  Implementing tasks...
     ✓ 1.1 Add theme context provider
     ✓ 1.2 Create toggle component
     ✓ 2.1 Add CSS variables
     ✓ 2.2 Wire up localStorage
     All tasks complete!

You: /opsx:archive
AI:  Archived to openspec/changes/archive/2026-01-23-add-dark-mode/
     Specs updated. Ready for the next feature.
```

## 使用建议（来自官方 Usage Notes）

- **选高推理模型**：官方推荐用 **Codex 5.5** 与 **Opus 4.7**（无论 propose 还是 apply 阶段）做规划与实现
- **保持 context 卫生**：进入 `/opsx:apply` 前清一次上下文窗口；规划与实现阶段尽量分会话，避免上下文污染
- **brownfield 增量建 spec**：不要一次性把整个老代码库写成 spec；只在动到某个 capability 时为它建/补 spec

## 安装与升级

```bash
npm install -g @fission-ai/openspec@latest   # 也支持 pnpm / yarn / bun / nix
cd your-project
openspec init                                 # 在仓库内生成 openspec/ 与 Agent 指令
openspec update                               # 升级后刷新 AI 指令文件
openspec config profile                       # 切换 default / expanded
```

## 与本知识库的关系

- 与 [[llm-knowledge-bases]] 同属「**文件驱动、跨会话持久上下文**」范式——规格 / 变更 / 任务全部留仓
- 与 [[agent-skills]] 互补：Skill 教 Agent *怎么做*（how），OpenSpec 记录系统 *应该做什么*（what）
- 与 [[trellis]]：OpenSpec ≈ Trellis 的 spec/task 子集 + 更轻的工作流；Trellis 在其上加了 workflow 状态机、sub-agent 分工、journal、平台适配
- 与 [[impeccable]] / [[open-design]]：那两者是垂直领域的 skill 包（命令规范、UI 设计），OpenSpec 在 Cursor / Claude Code 上可与它们并存
- 与 [[codegraph]]：codegraph 给 Agent 提供「现有代码长什么样」的索引，OpenSpec 给 Agent 提供「需求应该长什么样」的契约，二者从代码侧与意图侧夹击「AI 写错东西」

## 风险与局限

- **profile 选错会很别扭**：默认 `default` 只有三个命令，若直接给团队用容易缺少 `verify` / `bulk-archive`；建议团队项目一开始就切 `expanded`
- **opsx 命令名仍在演进**：v1.3 才把命名空间从 `/openspec:*` 改成 `/opsx:*`，旧文档和社区教程会混用，给新人造成混乱
- **依赖模型推理能力**：对低推理模型（小尺寸开源模型）效果显著劣化，官方明确推荐 Codex 5.5 / Opus 4.7
- **不强制工作流**：相比 [[trellis]] 用 hook 把 `<workflow-state>` 注入到每轮提示，OpenSpec 仍是文档约定型，AI 是否真的「先 propose 再 apply」很依赖用户自律与 system prompt

## 其他

- **遥测**：仅采集命令名与版本，CI 中自动关闭；opt-out：`export OPENSPEC_TELEMETRY=0` 或 `export DO_NOT_TRACK=1`
- **许可证**：MIT
- **AGENTS.md / MAINTAINERS.md**：仓库根目录的 Agent 指令与维护者清单，`openspec update` 会维护前者

## 相关链接

- 官网：[openspec.dev](https://openspec.dev/)
- GitHub：[Fission-AI/OpenSpec](https://github.com/Fission-AI/OpenSpec/)
- Discord：[社区](https://discord.gg/YctCnvvshC)
- 相关概念：[[llm-knowledge-bases]]、[[agent-skills]]、[[trellis]]
- 同生态：[[open-design]]、[[impeccable]]、[[codegraph]]
