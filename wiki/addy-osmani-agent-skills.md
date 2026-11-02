---
tags: [ai-agent, agent-skills, coding-agent, sdlc, code-review, open-source, workflow]
created: 2026-11-02
source: "raw/2026-11-02-addy-osmani-agent-skills.md"
---

# Addy Osmani Agent Skills

Addy Osmani 维护的 **生产级工程 Agent Skills 包**：把资深工程师在 Define → Plan → Build → Verify → Review → Ship 各阶段使用的工作流、质量门和反合理化纪律，写成 25 个可移植 `SKILL.md`，用 9 条阶段命令驱动，并带 4 个评审人格、7 份共享清单和仓库内三层 eval。MIT。协作者 Federico Bartoli、Joan León。

## 核心概念

- **阶段覆盖，而不是一条内循环**：元 skill `using-agent-skills` 把进来的工作路由到对应阶段；slash 命令与阶段一一对应
- **流程，不是散文**：每个 skill 是 Agent 要跟着走的步骤、检查点和退出条件，不是读完即可的参考文档
- **反合理化是标配**：每个 skill 都有 Common Rationalizations（Agent 用来跳步的借口及反驳）和 Red Flags
- **验证不可谈判**：结束时要有证据——测试通过、构建输出、运行时数据。「看起来对」不算完成
- **渐进披露**：`SKILL.md` 是入口，`references/` 里的清单按需加载，控制 token
- **人停在阶段之间，不停在每个任务之间**：`/build auto` 在计划被批准后自主实现全部任务；每个任务仍是测试驱动、单独提交，失败或高风险步骤会暂停
- **Google 工程文化写进步骤里**：Hyrum's Law、Beyoncé Rule、测试金字塔 80/15/5、约 100 行的 change sizing、Chesterton's Fence、主干开发、Shift Left、feature flag、把代码当负债的弃用流程。来源包括 *Software Engineering at Google* 与 Google engineering practices
- **不要同时开两个路由器**：可以单点借用别的包里的 skill，但两个 meta-skill 同时做路由会抢命令名、抢路由、混用不同的 TDD 哲学

## 九条命令

| 在做什么 | 命令 | 原则 |
|----------|------|------|
| 定义要做什么 | `/spec` | 先规格后代码 |
| 规划怎么做 | `/plan` | 小而原子的任务 |
| 增量实现 | `/build` | 一次一片 |
| 证明它能工作 | `/test` | 测试即证据 |
| 写下质量门槛 | `/constraints` | 决定一次，处处执行 |
| 合并前审查 | `/review` | 改善代码健康 |
| 审计 Web 性能 | `/webperf` | 先测量再优化 |
| 简化代码 | `/code-simplify` | 清晰优于机巧 |
| 发布到生产 | `/ship` | Faster is safer |

在 Claude Code 里，与内置命令重名时用命名空间形式，例如 `/agent-skills:review`。`/ship` 会并行拉起评审人格，再合成 go / no-go。

Skill 也会按正在做的事自动触发：设计 API 会带上 `api-and-interface-design`，做 UI 会带上 `frontend-ui-engineering`。

## 25 个 Skill

24 个生命周期 skill，加上元 skill `using-agent-skills`。

**Meta**：`using-agent-skills` — 把工作映射到正确 workflow，并定义共享操作规则。

**Define**

- `interview-me` — 一次只问一个问题，挖用户真正想要的，直到约 95% 置信；对应「interview me / grill me」
- `idea-refine` — 发散再收敛，把粗想法收成具体提案
- `spec-driven-development` — 写覆盖目标、命令、结构、代码风格、测试和边界的 PRD，然后再写代码
- `constraint-driven-development` — 访谈出质量门槛（带默认可接受阈值），写入 `CONSTRAINTS.md`，按成本放置检查，并抓住 Agent 关掉检查或跳过测试来刷绿

**Plan**：`planning-and-task-breakdown` — 把规格拆成带验收标准和依赖顺序的小任务。

**Build**

- `incremental-implementation` — 薄垂直切片：实现、测试、验证、提交；feature flag、安全默认、可回滚
- `test-driven-development` — Red-Green-Refactor、金字塔 80/15/5、DAMP 优于 DRY、Beyoncé Rule、浏览器测试
- `context-engineering` — 在对的时间喂对的信息：规则文件、上下文打包、MCP
- `source-driven-development` — 框架决策必须落到官方文档：核实、引用、标出未核实项
- `doubt-driven-development` — 对每个非平凡决策做对抗式新上下文审查：CLAIM → EXTRACT → DOUBT → RECONCILE → STOP；可选用用户授权的跨模型升级。用于生产、安全、不可逆或陌生代码
- `frontend-ui-engineering` — 组件架构、设计系统、状态、响应式、WCAG 2.1 AA
- `api-and-interface-design` — 契约优先、Hyrum's Law、One-Version Rule、错误语义、边界校验

**Verify**

- `browser-testing-with-devtools` — Chrome DevTools MCP：DOM、控制台、网络、性能
- `debugging-and-error-recovery` — 五步分诊：复现、定位、缩小、修复、加防护；停线规则与安全回退

**Review**

- `code-review-and-quality` — 五轴审查、约 100 行的变更规模、严重度 Nit / Optional / FYI、审查速度规范、拆分策略
- `code-simplification` — Chesterton's Fence、Rule of 500；降复杂度且行为必须不变
- `security-and-hardening` — OWASP Top 10、认证、密钥、依赖审计、三层边界
- `performance-optimization` — 先测量：Core Web Vitals、剖析、包体积、反模式

**Ship**

- `git-workflow-and-versioning` — 主干开发、原子提交、提交即存档点；任何代码变更都适用
- `ci-cd-and-automation` — Shift Left、Faster is Safer、feature flag、质量门、失败反馈环
- `deprecation-and-migration` — 代码即负债；强制弃用 vs 建议弃用；清僵尸代码
- `documentation-and-adrs` — ADR、API 文档、行内文档；写 *why*
- `observability-and-instrumentation` — 结构化日志、RED、OpenTelemetry、基于症状的告警；边做边埋点
- `shipping-and-launch` — 上线清单、flag 生命周期、分阶段发布、回滚、监控

## 人格、清单与 Skill 解剖

四个预置人格：`code-reviewer`（Staff 工程师，五轴，「Staff 会批准吗」）、`test-engineer`（Prove-It）、`security-auditor`（威胁建模与 OWASP）、`web-performance-auditor`（Core Web Vitals，Quick/Deep，指标诚实；经 `/webperf` 跑）。编排规则写在 `docs/agents.md`：人格不调用人格。

按需拉入的清单：`definition-of-done`（全项目常设门槛，对照单任务验收标准）、`testing-patterns`、`security-checklist`、`performance-checklist`、`accessibility-checklist`、`observability-checklist`、`orchestration-patterns`。

每个 `SKILL.md` 的固定解剖：frontmatter（`name` + `description`）→ Overview → When to Use → Process → Rationalizations → Red Flags → Verification。

## 仓库分层

可移植核心在 `skills/`（25）和 `agents/` + `references/`。宿主目录是各工具的发现约定，不能改名合并：

- Claude Code：`.claude/commands/`（9）、`.claude-plugin/`、`hooks/`
- Gemini CLI：`.gemini/commands/`
- Antigravity：`commands/`、`plugin.json`（部分版本里旧 TOML 包装命令不可发现，应直接调用带命名空间的 skill）
- Codex：`.codex-plugin/`、`.agents/plugins/`，直接消费 `skills/`
- Copilot CLI：根 `plugin.json`；生命周期 slash 命令不会出现在 Copilot 里，改为按 skill 名调用（如 `/spec-driven-development`）
- 贡献者工具：`scripts/`（13）、`evals/`（25 个用例）、`.github/workflows/`

Cursor 不随仓库带 `.cursor/`。工作流 skill 同步到 `.cursor/skills/`，短策略放 `.cursor/rules/*.mdc`，不要把完整 skill 贴进 rules。

## 安装

任意支持 [skills CLI](https://github.com/vercel-labs/skills) 的 Agent（文档称 70+，含 Claude Code、Cursor、Codex、Copilot、Cline）：

```bash
npx skills add addyosmani/agent-skills
npx skills add addyosmani/agent-skills --skill test-driven-development
```

单独安装一个 skill 时，只复制 `skills/<name>/`，**不带**仓库级 `references/`。skill 仍能跑，但指向共享清单的路径不可用。整库安装、clone，或把需要的清单拷进该 skill 自己的 `references/`。此缺口跟踪在 [#361](https://github.com/addyosmani/agent-skills/issues/361)。

Claude Code 推荐 marketplace：`/plugin marketplace add addyosmani/agent-skills`，然后 `/plugin install agent-skills@addy-agent-skills`。Marketplace 默认走 SSH；没有 SSH key 时用 HTTPS URL，或把 `git@github.com:` 改写为 `https://github.com/`。Codex CLI v0.122+ 用 `codex plugin marketplace add` / `codex plugin add agent-skills@agent-skills`，聊天里用 `@` 调用。已有代码库的推广路径见 `docs/adoption-guide.md`：绿地从第一天走完整生命周期，存量库则增量、验证优先。

## 三层 Eval

这是相对 [[matt-pocock-skills]] 和 Superpowers 的当前差异点，写在仓库里：

- **Tier 1** 查结构
- **Tier 2** 查 description 是否带用户真正会说的词，以及两个 skill 的路由是否冲突；确定性，跑在 CI
- **Tier 3** 用真实执行轨迹对照每个 skill 的期望打分

贡献标准：具体（可执行步骤）、可验证（带证据的退出条件）、经过实战、最小。

## 和 Superpowers、Matt Pocock Skills 怎么选

三者优化的时刻不同。对照见仓库 `docs/comparison.md`。

| | 本包 | Superpowers（obra） | [[matt-pocock-skills]] |
|--|------|---------------------|------------------------|
| 组织方式 | SDLC 阶段 + 元 skill 路由 | 一条纪律循环：brainstorm → plan → execute → review | 可组合命令工具箱 |
| 规模（文中口径） | 25 | 约 14，深在内循环 | 约 30，含 in-progress / deprecated |
| 独特机制 | 反合理化表、并行评审人格、清单、三层 eval | 子 Agent 执行 + 任务评审员、worktree 隔离、先写失败测试再写 skill | grilling（一次一问、走设计树）、user-invoked / model-invoked 拆分 |
| 适合 | 一个功能从前到后，每阶段有人检查 | 长时间自主、重推理、探索性工作 | 日常短循环，尤其是先把需求问清 |

Om Mishra 的单任务对照（同一模型、同一仓库、同一 prompt，只换 skill 框架，不是基准）：本包更快进入写代码（约 8 分钟 vs 约 12 分钟），验证轮次更多（7 vs 5，含全量测试），并抓到功能测试之外的兼容问题；Superpowers 把更多预算花在事前架构推理，作者仍把它当演进中生产系统和无既定模式的探索工作的日常驱动。Token 效率基本相同，两边都重规划了一次。取舍是 **广而有纪律的验证** 对 **重的事前推理**。

小改动可以跳过前面阶段，直接 `/test` 和 `/review`。需求拷问上，Pocock 的 grilling 是参照实现，本包的 `interview-me` 精神相近，并在加可选的协作模式。团队更适合命名阶段、人格、清单和会让 CI 失败的 eval；个人则选和自己默认工作方式一致、仪式最轻的那个。

三者都还没解决 **跨会话持久记忆**：一次会话里学到的东西很少干净地带到下一次。可以单点混用（例如借 Pocock 的 `grill-me`），不要让两个包同时当活跃路由器。

## 相关链接

- [[agent-skills]] — Skill 规范、渐进式加载与设计模式；本仓库是按该形态分发的一整包工程纪律
- [[matt-pocock-skills]] — 另一套可组合工程 skill；grilling / TDD 更尖，生命周期在 Build 之后更薄，仓库内无 eval
- [[openspec]] — 仓库内活规格与变更提案；本包的 `/spec` 是 Agent 工作流，不是独立规划层
- [[trellis]] — 团队级脚手架把 spec / task 固化进 git；本包把阶段纪律装进 skill
- [[ponytail]] — 压实现层代码量；本包的 `code-simplification` 要求行为不变
- [[impeccable]] — 设计质量注入；本包的 `frontend-ui-engineering` 覆盖组件、设计系统与 WCAG，不替代设计命令集
- [[reviewbench]] — 用真实 PR 评估审查 Agent；本包用仓库内路由与行为 eval 约束 skill 本身
- [[codegraph]] — 给编码 Agent 的符号与调用链索引；本包的 `context-engineering` 管的是会话里喂什么

## 参考

- 仓库: https://github.com/addyosmani/agent-skills
- 对比说明: https://github.com/addyosmani/agent-skills/blob/main/docs/comparison.md
- skills CLI: https://github.com/vercel-labs/skills
- Superpowers: https://github.com/obra/superpowers
- 对照实验: https://www.linkedin.com/pulse/superpowers-vs-agent-skills-faster-shipping-safer-reasoning-om-mishra-dzakf/
