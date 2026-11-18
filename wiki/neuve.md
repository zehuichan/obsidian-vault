---
tags: [ai-agent, code-review, human-in-the-loop, git, mcp, typescript, open-source, review-desk]
created: 2026-11-18
source: "raw/2026-11-18-neuve-code-review.md"
---

# Neuve

Neuve 是 NeuveAI 开源的**面向 Agent 时代的代码评审工具**：把一段 commit range 的变更拆成「评审单元」，按复杂度 × 可验证性分到四条 lane（自动化 / Agentic / 建议人工 / 必须人工），让人只看真正需要人判断的那部分；评审在本机跑，复用你已有的 Claude Code / Codex / API 账号。

> 官方定位：an open-source code review tool that allows developers and agents ship code with more confidence · 口号 *Large code changes never felt so small* · 仓库 [NeuveAI/neuve](https://github.com/NeuveAI/neuve) · 主页 [neuve.ai](https://neuve.ai) · MIT · TypeScript · 剪藏时 3 stars（2026-11-16 创建，极早期）

README 自己的提示：*Neuve is early. Expect rough edges and changing commands.*

## 核心概念

- **要解决的问题**：作者做 QA Agent 时发现最大摩擦不在 QA，而在更早的 SDLC——界定范围、确认做的是对的事、事后审计评审。Agent 让「done」很容易，难的是到「good」；大变更不该压垮团队
- **四条 Lane**（责任类别）：
  - **Automation**：CI/CD 已覆盖、复杂度低
  - **Agentic**：容易验证，或复杂度不值得人看
  - **Human Suggested**：超出可安全交给 Agent 的范围，但还不必立刻人工；Agent 先出处置，人可附议或推翻
  - **Human Required**：难以用 Agent 验证、复杂度高或上下文信号强；Agent 只准备**建议评审**，**永远不批准**
- **Batch → Review unit**：一个 commit range 由 harness 切成一个或多个 **batch**（各管一个行为/义务）；batch 内按职责把变更行范围聚成 **review unit**（同一 lane 内，一个文件的变更算一个 unit）。commit 不等于批准，只有记录下来的 **disposition** 才算
- **Review desk**：人类的评审界面，替代 PR 页面——batch 概览 + 逐 unit 评审台
- **范围不变性**：Git 的 range 不可变，所以同一 range 的计算结果可内容寻址复用，不必重算
- **本机运行、暂不回帖**：目前不向 GitHub 发评论或 review；结果只在本机的 review desk 里

## 详细内容

### 评审流水线

README 的一行概括：`commit range → goal/batches/parts → four agents look for problems → each finding checked again → you record your review in the review desk`。

架构记录 0005 把它写成固定形状的试验管线：**Frame → Map → 四个 finder → Verify → code fill → Link → Publish**，工作有界、range 全覆盖。

| 步骤 | 做什么 |
| --- | --- |
| Goal / batches / parts | 从能取到的 ticket 或 plan 里陈述变更的目标与「做完」的含义，没有就明说；每一行变更属于且仅属于一个 part，即便运行被截断 |
| 四个 finder | **Scope fit**：是否满足「做完」的定义，或必须由人判断；**Correctness**：沿变更代码追到一个能看见的失败；**Risky edges**：数据、鉴权，以及导出/配置/wire format 等接口；**Tests**：测试或其他证据是否证明了「做完」 |
| Checking | 检查针对 **Neuve 自己的代码收集的证据**，**不给模型查询工具**；无法核实的 finding 只计数，**不当评论展示** |
| 你来决定 | 从 batch 概览打开某个 part，已核实的 finding 贴在对应行旁，你在 review desk 里记录评审 |

### Lane 矩阵与「校验要求」

起始象限（可被历史、旧 bug、故障、blast radius 抬高，**只升不降**）：

| | 易验证 | 难验证 |
| --- | --- | --- |
| **低复杂度** | 静态自动化或 Agentic | Human-suggested |
| **高复杂度** | Agentic 或 Human-suggested | Human-required |

- 评估是「尽力而为」：有缺口、无法自信判断时，unit 进 Human-suggested，且不加特殊标签；权重与阈值尚未设定
- **Validation requirement**：确认 unit 行为正确必须观察到什么——来自范围里的「Done when」或由变更隐含。证明靠**观察到的证据**：浏览器侧为服务端/浏览器/网络日志、截图与视频（词汇表写明经 Chrome DevTools MCP 采集），服务端为遥测与日志；**各类证据必须一致**，单一种类不够；自定义脚本不算证据；测试是底线要求，但「生成代码里测试很多」是警告信号，伪造的测试或证据本身就是缺陷
- **Checkpoint**：校验要求在评审者眼中的样子，带证明状态（按优先级）：**Missing**（无记录）→ **Out of date**（依赖的记录已不新鲜）→ **Conflicting**（记录互相矛盾）→ **Proven**（≥2 种证据、适用种类齐全且一致，能测则含最新测试）→ **Thin**（其余）。Missing / Out of date / Conflicting 视为未满足，Thin 不算
- **Blast radius**：被调用方、依赖链接、所在依赖图连通组大小等**分项实测**，不合成单一分数；没测到的部分显式标为缺失，不当作 0；只能抬高 lane，不会降低

### Review desk 界面

- **Unit rail**（按 lane 分组的 unit 列表，含 human-gate 计数）、**unit 的 contextual briefing**（带短语链接与引用 chip）、**diff**、**Context margin**（边栏 marker pin，上下文不进代码，要才展开）、**Decision dock**（记录处置）
- Marker 披露四层：**L0 pin → L1 peek → L2 context card → L3 source sheet**，代码本身不位移
- Decision dock 因 lane 而异：human-required 可批准/请求修改；human-suggested 可「agree」或覆盖；**agentic 不能由人批准**（批准是 Agent 的），只能委托/反驳/请求修改/追加评论；static automation 只显示红绿状态
- **Review delivery** 四档（提议名 Facts / Briefed / Explained / Guided）：同一批有引用的事实，档位越高解释越多；最高档下 diff 让位于 ticket 的「Done when」，human-required 工作还会给出 **suggested review**。它**不改变事实、lane、dock 动作或谁来决定**
- **Reviewer's tax** 是核心度量思想：评审者为重建信息、反复处理而花的精力；主要度量为不必要的重复、错误归因、缺失归因

### 架构（六层，目标态）

| 层 | 职责 |
| --- | --- |
| **Harness** | 终评流水线：范围收集 → 混合检索 → 精炼重排 → 确定性检查 → lane 矩阵 → 预生成处置；作为 Records 的客户端运行 |
| **System of records** | 所有进程链接的库，一份 SQLite 位于仓库 `.git/neuve/`：append-only 的批次/单元/上下文引用/评估/处置/评论/周期/Agent 身份与活动，外加内容寻址的计算存储与事件队列 |
| **Graph** | 语义层：索引代码与本地文档并保存 embedding 与出处元数据，随 commit 增量更新；每台机器一个共享实例。是 [Entire Graph](https://github.com/entireio/entire-graph) 的 fork |
| **CLI** | 给内/外环 Agent 评审者用：记录工作、证据、身份、下钻数据；负责安装、hooks、本地开发命令 |
| **MCP** | 经 stdio 向编码 Agent 暴露工具，随会话生灭，不起常驻端口 |
| **Web app** | 事件驱动的人用界面（batch 概览 + review desk），BFF 只投影与转发 |

工作流：**Capture**（git hook / Agent hook 写 spool 即返回）→ **Queue**（Records 转成后台任务：embedding、图更新、评审流水线）→ **Work**（按需起 worker，空闲退出，launchd 兜底；无常驻服务）→ **Reuse**（按输入哈希查缓存，range 不可变所以不重算）→ **Review**（写出 batch 包，Web/CLI 展示，处置作为记录追加）→ **Cycle**（所有需要的处置齐了即结束，评论成为下一周期上下文）。

其他架构决定（节选自 17 条 ADR）：无认证，Agent 以 fingerprint + alias 识别，git 凭据即可（0009）；Zod 为唯一 schema 库（0011）；完整会话数据保留 30 天（0004）；Harness 为 Ace 风格任务，跑在 Effect + Bun 上，用 pi 对话与隔离的 Claude Code 循环（0014）；ACP 只是连接 Codex / Claude Code 的通道，**终评永不依赖它**。架构文档明确说：**这些记录描述目标架构，当前代码很多地方尚未对齐**，由「修正里程碑」去补。

### 安装与使用

```sh
curl -fsSL https://neuve.ai/install.sh | sh     # 目前仅 Apple silicon Mac；Linux/Windows 构建在做
cd /path/to/your/repository
neuve setup                                      # 选择模型路线
neuve review main..my-branch                     # 后台 worker 跑评审，关终端不中断
neuve review desk                                # 本机 http://127.0.0.1:4141/
neuve doctor                                     # 检查缺什么、怎么修
neuve upgrade                                    # 升级到最新 release
```

**三种模型路线**（`review.route`，设置在 `~/Library/Application Support/Neuve/settings.json` 或 `~/.config/neuve/settings.json`）：

| route | 模型 | 前提 |
| --- | --- | --- |
| `anthropic`（默认） | Claude Opus 5.5 经 Anthropic API | 环境变量 `ANTHROPIC_API_KEY`（必须是 API key，订阅 token 不行） |
| `openai` | GPT-6.1 Sol，走 ChatGPT 订阅 | `neuve auth login openai` |
| `claude` | Claude Opus 5.5，走 Claude 订阅 | 已装 Claude Code 并 `claude auth login`；Neuve **不读取**该登录态 |

`review.effort` 可选 `low` / `medium`（默认）/ `high`。

**采集 commit 与 Agent 会话**：`neuve setup hooks` 给仓库装 `post-commit` / `post-rewrite`（链式保留旧 hook 为 `.pre-neuve`），并向用户级配置写入 Claude Code 与 Codex hooks（`~/.claude/settings.json`、`~/.codex/hooks.json`，先备份）；hook 永远 exit 0、不向 Agent 上下文加内容、不阻塞提交；commit 通过进程祖先关系或 harness 的 session 变量关联到产生它的 Agent 会话。`neuve mcp` 提供 `vent`（Agent 每会话一次说出自己遇到的困难，不改变任务或评审）与 `record_proof`（记录观测到的证据）。其他命令：`neuve skills`（装教 Claude Code/Codex 用 Neuve 的 skill）、`proof`、`worker`、`jobs`、`relations`、`review show`、`agents`、`records prune`。

## 与本知识库的关系

- 与 [[reviewbench]]：ReviewBench 是评**审查 Agent 找问题能力**的离线基准（precision / recall）；Neuve 是**产品化的评审工作流**，其「找不到证据就不展示」和「finding 必须被代码收集的证据再核实」直接对应基准里 false positive 的治理思路。仓库本身没提它们的关系，这是读后的对照
- 与 [[harness-design-long-running-apps]] / [[demystifying-agent-evals]]：同样是「生成 vs 评估分离、评估要有证据」；Neuve 把评估器做成面向人类的 lane 分流与 checkpoint 证明状态
- 与 [[codegraph]]：二者都用预索引的代码图给 Agent 提供调用链/依赖上下文；Neuve 的 Graph 层更偏评审（blast radius、出处元数据、增量按 commit 更新）
- 与 [[trellis]]：都把过程记录固化在仓库内（Trellis 是 git Markdown，Neuve 是 `.git/neuve/` 里的 append-only SQLite）；Trellis 管开发前的 spec/task，Neuve 管开发后的评审
- 与 [[openspec]]：Neuve 的 goal / 「Done when」从 ticket 或 plan 取，计划层可由 OpenSpec 一类规格流程提供
- 与 [[dsh-superagent]]：两者都把「少占人的注意力」当硬约束——DeepSeek 用授权与面板，Neuve 用 lane 与 reviewer's tax
- 与 [[agent-browser]] / [[e2e-testing-framework]]：浏览器侧的日志、截图、视频与测试运行是 Neuve 的「观测证据」来源，agent-browser 一类 CLI 可作为采集手段
- 与 [[effective-context-engineering]]：Neuve 的 query context budget、digest、按需披露（L0–L3）是在评审场景下对「上下文是有限预算」的具体落地
- 与 [[pi]] / [[agent-skills]]：Harness 用 pi 对话；还通过 skill 教 Claude Code / Codex 使用 Neuve

## 风险与局限

- **极早期**：仓库 2026-11-16 才创建，README 自述「rough edges and changing commands」；架构文档承认目标态与当前代码有差距，且 design decisions 会覆盖旧文档
- **平台受限**：一行安装仅 Apple silicon Mac；Linux / Windows 构建未就绪（SETUP 中的 `~/.config/neuve` 只表示设置路径，不代表已支持）
- **不回帖 GitHub**：目前无法融入现有 PR 流程，结果只在本机 desk 里
- **改写用户级配置**：`neuve setup hooks` 会改 `~/.claude/settings.json` 和 `~/.codex/hooks.json`（有备份与 `--dry-run`）；原始会话转录（含 prompt 与工具输出）会被拷进本地 store，保留 30 天后清除，要注意敏感内容
- **成本与模型绑定**：评审跑在你自己的账号上，默认走 Anthropic API 计费；订阅路线依赖本机 Claude Code / ChatGPT 登录
- **阈值未定**：lane 的权重与阈值、blast radius 的连通组大小等仍标注「尚未设定/未测量」；评估是 best effort，不确定时降级到人工建议
- **证据质量未知**：仓库未给出与 [[reviewbench]] 这类基准的对比分数，lane 分流是否可靠需要自己在真实仓库上验证

## 相关链接

- 仓库：[github.com/NeuveAI/neuve](https://github.com/NeuveAI/neuve) · 主页：[neuve.ai](https://neuve.ai)
- 文档：[CLI guide](https://github.com/NeuveAI/neuve/blob/main/apps/neuve-cli/SETUP.md)、[Architecture](https://github.com/NeuveAI/neuve/blob/main/architecture/README.md)、[Product vocabulary](https://github.com/NeuveAI/neuve/blob/main/vocabulary.md)
- 致谢项目：GitHub Next [Ace](https://github.com/githubnext/ace)（`@ace/host` / `@ace/channel` 改编）、[Entire Graph](https://github.com/entireio/entire-graph)（Graph 层 fork）、Pierre [`@pierre/diffs`](https://github.com/pierrecomputer/pierre)（diff 渲染，Apache-2.0）
- 原始剪藏：`raw/2026-11-18-neuve-code-review.md`（含 README 与三份子文档原文）
- 同类对照：[[reviewbench]]、[[codegraph]]、[[trellis]]、[[openspec]]、[[dsh-superagent]]、[[harness-design-long-running-apps]]、[[demystifying-agent-evals]]
- 工具层相关：[[agent-browser]]、[[e2e-testing-framework]]、[[effective-context-engineering]]、[[pi]]、[[agent-skills]]
