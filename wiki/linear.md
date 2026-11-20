---
tags: [project-management, issue-tracking, ai-agent, mcp, code-review, saas, automation]
created: 2026-11-20
source: "raw/2026-11-20-linear-homepage.md"
---

# Linear

Linear 是一款面向产品团队的**项目/Issue 管理与产品开发系统**，官网（`linear.app/homepage`）当前把自己定位为 *The product development system for teams and agents*：人和 Agent 在同一套 issue、计划与评审流程里协作，从起草 PRD 一直到推 PR。

> 官方口号：*Purpose-built for planning and building products. Designed for the AI era.* · 首页自称「powers over 50,000 product teams」· 本文来自首页营销页，**不含定价、权限、安全、API 细节**；页面里的 issue / 代码 / 聊天均为演示数据。

## 核心概念

- **三个卖点（Fig 0.1–0.3）**：
  - **Purpose-built**：由顶尖产品团队的实践与原则塑造
  - **Powered by agents**：为人与 Agent 共享的工作流而设计，「从起草 PRD 到推 PR」
  - **Designed for speed**：减少噪音、恢复节奏，让团队高速且专注地交付
- **Agent 是队友**：能被 `@Linear` 提及、被指派/委托整个 issue，端到端产出（演示里是改 2 个文件并开 Draft PR 等人评审）
- **Loops**：「Loops automate product work by responding to changes in Linear and connected tools」——对 Linear 与已连接工具里的变化做出响应的自动化；可从 Slack 与 GitHub 触发、可作为模板分享、可自定义 reasoning level（首页顶部有「New Loops」入口）
- **Diffs / Reviews**：Linear 自带的代码评审体验（Diffs）与评审列表（Reviews）；评审时 PR 与对应 issue、project 上下文并列展示
- **Triage Intelligence**：演示里自动给 issue 打 `Performance`、`iOS` 标签
- **面向 Agent 的接入面**：官网直接给 Agent 留话——连接 **Linear MCP server** `https://mcp.linear.app/mcp` 可读写 issues / projects / documents；配置指南 `linear.app/docs/mcp.md`；全站文档索引 `linear.app/llms.txt`

## 详细内容

### 首屏演示：一条 issue 的 Agent 生命周期

演示 issue `DRV-8852 Faster app launch`（共 84 条里的第 1 条）：描述是「iOS 启动时，最小必要状态就绪就先渲染 UI，不再阻塞在完整的 `vehicle_state` 同步上」。Activity 时间线串起整条链：

1. **Linear 经 Slack 代 Karri 创建 issue**（来源是聊天，不是手填）
2. **Triage Intelligence 自动加标签** Performance、iOS
3. 人在评论里描述现象（「一直转圈，像车消失了」）
4. Jori 评论 `@Linear can you take a stab at this?`——**把任务交给 Agent**
5. Agent 接手后：改了 2 个文件、开出 **Draft PR 等你评审**、issue 自动从 **Todo → In Progress**

属性栏显示：状态 In Progress、优先级 Medium、负责人 Karri、`Linear`（Agent 同时出现在属性里）、Cycle 144、Labels、Releases（Linear App 1.6881…，Jun 25）。

### 四个产品面

| 模块 | 官方描述 | 页面里的演示 |
| --- | --- | --- |
| **Intake and integrations** | 把对话与客户反馈自动转成可执行 issue，即时路由、打标签、定优先级到正确团队 | Slack `#product` 线程里三人讨论 iOS 冷启动慢，最后 `@Linear create issues and assign to me`；列表视图按 Backlog 8 / Todo 71 / In Progress 3 / Done 53 分组 |
| **Planning and monitoring** | 从想法到上线的规划与导航；用 initiatives、战略 roadmap 与保持更新的 PRD 对齐团队 | （首页无独立演示） |
| **AI and automations** | 构建并部署与你并肩的 AI Agent；复杂任务可协作，也可把整个 issue 委托出去端到端完成 | 即上面的 DRV-8852 |
| **Build, review, and ship** | 用更清晰的 diff、更好的上下文、更少的来回评论简化 code review，在不牺牲质量下让 PR 持续流动 | 看板 In Review 3 / In Progress 4 / Todo 4，issue 旁带 PR 号与「Working…」状态；一段 `HomeScreen.tsx` 的 diff（`isFullySynced` 布尔 → `SyncStatus` 枚举） |

演示里的最后一段代码是个小笑话式的 JSX：`<Diff.Provider>` 下依次是 `Slow` / `Fragmented` / `HumanOnly`（现状痛点）与 `Frictionless` / `Integrated` / `AgentReady`（目标）——**这是我对这段代码寓意的解读**，页面没有文字解释，但与「评审要快、要集成、要对 Agent 友好」的文案一致。

### Changelog 里的最近信号（2026）

| 日期 | 更新 |
| --- | --- |
| Nov 12 | **BYOK**：可给 Linear 的**云端 coding agent** 自带模型 key，使用已有的 provider 额度与议价费率 |
| Nov 4 | **Mobile Diffs**：Diffs 评审体验登陆 iOS 与 Android |
| Oct 22 | **Slack 与 GitHub 的 Loop 触发器**：Loops 可由 Slack / GitHub 触发、做成模板分享、自定义 reasoning level |
| Oct 8 | **Stacked pull requests**：Diffs 里按栈浏览，一眼看 review / CI / merge 状态；相关 PR 在 Reviews 中折叠成一行，并带 issue 与 project 上下文 |

由此可见 Linear 当前的投入方向：**评审体验（Diffs / Reviews / 栈式 PR / 移动端）+ 自动化（Loops）+ 云端 coding agent（含 BYOK）**。

### 社会证明

- Gabriel Peal（OpenAI，Staff Software Engineer）：「用 Linear 的工艺感会潜移默化让你做出更好的产品」
- Nik Koblov（Ramp，Head of Engineering）：「我们的速度很猛，Linear 帮我们保持 action biased」
- 「over 50,000 product teams」——自述数字，页面未给出口径

## 与本知识库的关系

以下为读后的对照，Linear 页面本身没有提到这些项目：

- 与 [[neuve]]：两者都在回答「Agent 让产出变快后，评审怎么不成为瓶颈」。Linear 的路线是**把评审放进 issue tracker**（Diffs / Reviews + issue/project 上下文），Neuve 是**本机、仓库内的 review desk + lane 分流**。Neuve 的 goal / 「Done when」要从 ticket 或 plan 取，Linear 的 issue / PRD 正是这类来源（经 MCP 可读），但二者是否有集成属推断
- 与 [[dsh-superagent]]：同样是「监听 GitHub / 聊天、把事务转成 ticket、让 Agent 主动推进」；DeepSeek 用 Mandate / 授权约束主动性，Linear 的对应机制从首页看不出来，需读 Agents / Loops 文档
- 与 [[design-agent]]：交互形态相似——在 Slack 线程里 `@` 一个 Agent 就把工作启动起来
- 与 [[trellis]] / [[openspec]]：这两者把 spec / task 固化在**仓库内的 Markdown**，Linear 把计划与任务放在**托管的协作系统**里；两条路线在「Agent 读什么上下文」上形成对照
- 与 [[reviewbench]]：若 Linear 的云端 agent 或评审功能要评估「找问题的质量」，这类离线基准是现成参照
- 与 [[agent-skills]]：Linear 通过 MCP + `llms.txt` 面向 Agent 暴露能力，和 skill 的渐进式披露思路类似——先给 Agent 一个入口索引

## 风险与局限

- **纯营销页**：只有文案和演示，没有价格、套餐、权限模型、数据留存、SOC2 等信息；「50,000 teams」「high velocity」等均为自述
- **演示数据不是真实能力证明**：issue 编号、PR 号、日期（Oct 2026）、`Upgrade to Claude Opus 5.5` 这类 issue 都是示例内容，不代表产品支持的模型清单
- **关键功能细节缺失**：Loops 的触发条件与权限、云端 coding agent 的沙箱与计费、Diffs 的评审流程、Triage Intelligence 的规则等都只有一句话，需看各自的 Learn more / docs
- **Agent 自动改动的治理**：首页展示「`@Linear` 即开 Draft PR」，但谁能委托、能改哪些仓库、如何审计页面没说；落地前应单独核实
- **剪藏可能是缓存内容**：WebFetch 可能返回缓存，Changelog 以 2026-11-20 抓到的为准

## 后续可补充的素材

- `https://linear.app/docs/mcp.md`（MCP 配置）与 `https://linear.app/llms.txt`（文档索引）——页面主动推荐给 Agent 的入口
- Loops、Diffs、云端 coding agent 的详细文档

## 相关链接

- 官网首页：[linear.app/homepage](https://linear.app/homepage)
- Agent 入口：[Linear MCP](https://mcp.linear.app/mcp) · [MCP 指南](https://linear.app/docs/mcp.md) · [llms.txt](https://linear.app/llms.txt)
- 原始剪藏：`raw/2026-11-20-linear-homepage.md`
- 同类/对照：[[neuve]]、[[dsh-superagent]]、[[design-agent]]、[[trellis]]、[[openspec]]、[[reviewbench]]、[[agent-skills]]
