---
tags: [agent, design, langchain, deep-agents, managed-deep-agents, slack, hyperframes, video, harness, sandbox, open-source]
created: 2026-10-29
source: "https://github.com/langchain-ai/design-agent（raw/2026-10-29-langchain-design-agent.md）"
---

# Design Agent（LangChain Content Studio）

LangChain 开源的「设计 Agent」：把团队里反复出现的设计需求（缩放广告图、卡片、幻灯片、图表、示意图、发布短片）变成在 Slack 线程里对话即可交付的 **有品牌约束、经代码校验的成品文件**；它是 LangChain 内部在跑的 **Content Studio** 的开源版，基于 Deep Agents，部署在 Managed Deep Agents（MDA）上，Apache-2.0。

## 核心概念

- **模型管创意，代码管交付**：模型做创意判断；渲染、度量、闸门检查、文件投递、权限、报告里的每个数字都由代码负责。
- **每个线程一台「设计师的电脑」**：独立 LangSmith 沙箱（Linux + Chrome Headless Shell + FFmpeg + HyperFrames + D2 + ECharts + sharp + 字体），产出物全是项目文件夹里的代码——一张幻灯片是 HTML，一部短片是动画时间线，改动 = 编辑 + 重渲染，而非重来。
- **品牌即代码**：布局只接受品牌 token（颜色、字体、间距）；品牌事实只来自 brand pack，凭 `brand-lint` 等检查卡口。
- **看自己的作品**：渲染后用视觉 `look` 工具亲自看图，再决定能不能发。
- **小 prompt + 按需 skill + 设计 wiki**：常驻 prompt 约 4k tokens；先读 router skill 选工作流，35 个运行时 skills 按需加载；wiki 放手艺（排版、色彩、版式、图表），brand pack 放事实。
- **密钥不进沙箱**：沙箱无凭证、默认无网络；图像/语音/音乐生成、素材抓取、Paper、MCP、投递等都是宿主工具，结果以文件形式落入沙箱。
- **异步 helper + 唤醒**：主 Agent（lead）规划并把长任务派给后台 helper，自己立刻空出来继续对话；helper 完成/失败/卡住时由 harness 唤醒 lead。
- **持续学习**：作业结束后 `learn_from_job` 对所用 skill 写观察笔记，后续作业读取并打分，不靠谱的被剪掉。

## 详细内容

### 它能做什么、怎么触达

- 触达面：**Slack**（@提及 + 附件；同一线程内单条消息分步更新、页脚显示运行中任务与累计成本、中途追加说明会作用于进行中的工作）、HTTP API、LangSmith Studio、终端 `design-agent ask`。
- 交付物：缩放广告、LinkedIn 卡片、幻灯片、数据图表、示意图、缩略图、GIF 编辑、解说视频、发布短片等。
- 成绩（官方自述）：自 8 月起交付 3000+ 资产；相关帖子在 X 上约 450 万次浏览；四分之三请求来自市场部之外；中位请求约 0.5 美元模型花费，长片与长 deck 更贵。
- 演示片：让它为自己做「发布短片」（Rocky 风格，47 秒 1-bit 小鸡到冠军），以及用「自然纪录片 × 像素鹦鹉」解释自己的 harness，均由 Agent 在一个 Slack 线程内分镜、制作、旁白完成。

### 架构：一份 assembly，多种 runtime

`src/design_agent/assembly.py` 是唯一的 Agent 装配（模型、工具、中间件、子 Agent、策略），同时服务 MDA、CLI、自托管 API 与 Slack；不同之处由 runtime profile 提供（沙箱后端、文件投递、唤醒方式、状态持久化）。

| | Managed Deep Agents | 自托管 |
| --- | --- | --- |
| 线程与记忆 | LangSmith；团队/个人两层记忆 | Postgres checkpointer + store，无记忆分层 |
| 沙箱 | LangSmith 沙箱（16 GiB 盘 / 8 vCPU，默认拒绝出站） | Docker 容器（无网络、丢弃 capabilities、非特权用户） |
| Slack | 托管 Slack app | 自建 Bolt app（Socket Mode 或签名 HTTP） |
| 唤醒与维护 | 运行时一次性 schedule；`housekeeping` 每 15 分钟 | 进程内 waker + 维护循环 |
| Skills | 同步到 Context Hub 只读挂载 | 从 checkout 读取 |

请求路径：消息进入（Slack/API/CLI）→ lead 一轮：把 skill 脚本、检查器、brand pack、字体按 sha256 清单只读地 stage 进线程沙箱 → 规划、编排、渲染、`look`、跑闸门 → `upload_files` 交付并记入项目 `DELIVERY.json`（防重复发送）。

信任边界：模型输出、用户文件、Slack 事件、画布内容、MCP 元数据、网页都是**数据**，不授予权限；**skill 引导判断，代码授予能力**——改 skill 不能暴露工具或放宽策略。

### 沙箱配方

一份 `sandbox/setup.sh` 按版本与 sha256 钉死 Chrome Headless Shell、FFmpeg、HyperFrames、sharp、ECharts、D2、开源字体，末尾用探针渲染一页、一部片、一张图；同一配方生成：本地/自托管 Docker 镜像、MDA 每线程克隆的 LangSmith 快照、CI 镜像。文件持久化在 `/workspace/projects/`。

### 检查闸门（代码可数的事交给代码）

| 检查 | 抓什么 |
| --- | --- |
| `brand-lint` | 偏离品牌的颜色、字体、圆角、文案 |
| `video-gate` / `hyperframes check` | 非确定性动画、布局、动效与对比度问题 |
| `artifact-contract` | 尺寸/时长不对、文件缺失、首帧空白 |
| 字体检查 | 品牌字体悄悄回退（含 web fonts） |
| 图表/图结构契约 | 截断基线、混用单位、占比不为 100%、图结构被改变 |

「品味」仍留给设计师；设计师的判断通过 skill 与检查沉淀下来。

### 编排 harness（`src/design_agent/harness/`）

harness 只依赖标准库、`langchain`、`langchain_core`、`langgraph`，通过三个小协议接入宿主，因此同一套原语可跑在 MDA、自托管、普通 Deep Agents 图与单元测试里：

- `HelperRunner`（`start` 立即返回 task id）· `SandboxExec`（沙箱执行）· `Waker`（`wake` / `schedule` / `cancel`）。
- **事件脊柱**：所有后台动静都是 `Event`（`task.done`、`watch.stalled`、`wakeup.due`…），经进程内总线发布，由 inbox、成本计量、可存储的 `EventLog`（保留一周）订阅。
- **注意力规则**：`NOW`（helper 完成/失败/中断、watch 失败/卡住/结束、check-in 到期、需要人处理）→ 唤醒 lead；`NEXT`（进度笔记、流水线步骤）→ 搭下一轮顺风车；`LOG`（启动、逐帧进度、用量）→ 仅记录。未知事件默认唤醒，不静默丢弃。
- **唤醒合并与重试**：5 秒内的 `NOW` 事件合成一条 notice；唤醒只有收到 `WakeReceipt`（lead 的 run 已启动）才算送达，失败按 10s / 1min / 5min 重试，之后留给 lead 的下一轮；带 `dedupe_key` 防重复。回复恰为 `[quiet]` 表示无话可说。
- **MDA 上的限制**：schedule 的 `at` 至少一分钟后并向上取整，唤醒延迟约 60–120 秒；只有 run 自己的运行时才能为该线程建 schedule；用「哨兵 schedule」（提前 15 分钟、每 5 分钟推后）应对进程重启；每个 owner 最多 20 个未过期 schedule。上游诉求：公开的「立即在此线程启动 run」API。
- **后台 helper 与重启**：helper 是 asyncio 任务；ledger 每分钟写心跳，超过 3 分钟无心跳的被判为 `task.interrupted`。
- **可恢复 helper**：`TranscriptCheckpointMiddleware` 边做边存历史，`resume_task(id)` 恢复；存下的内容保证可回放——图片转文本桩、丢弃 thinking 块、被切断的工具调用补「效果未知」结果、超 8 MB 丢最旧步骤。
- **流水线即代码**：`WorkflowRunner` 里控制流是 Python，每个创意调用是 helper brief；`ctx.agent / parallel / pipeline / ask`；每步按 brief+effort+schema+label 的哈希 memoize，重跑只补未完成步骤；要求流水线确定性（无时钟、随机、环境）。
- **并行 looks**：为每个创意方向复制项目到 `looks/<name>/`，一个 helper 一个方向，让人挑选后合并（只在静帧阶段做，不做整片渲染）。
- **渲染 watcher**：每 15 秒轮询，解析 `hyperframes`（`@hf-progress` 机器行）、`ffmpeg`、`json`、`regex`、`heartbeat` 进度；5 分钟无进展报 `stalled`；`nohup/setsid` 脱管渲染，以 `.exit` 退出码为裁决；故意忽略泛 `Error` 文本，因为 best-effort 渲染会把页面脚本错误当作警告。
- **上下文与成本**：统一 `cache_control` TTL（lead 1h、helper 5m）、`StaggerGate` 让同批 helper 共享前缀缓存、按真实成本计图像 token、`meter.py` 按来源/角色计价（价格在 `config/prices.toml`）。
- **学习**：`ObservationStore` 近重复合并（Jaccard ≥ 0.8 / difflib ≥ 0.85）、helpful/harmful 投票、harmful 领先 2 票即剪；渲染给下个作业时标明是「不可信笔记」，只权衡、不当规则。

### 模型

默认 Claude Opus 5.5；任何带视觉的 tool-calling 模型可用，也支持 OpenAI/Google、任意 OpenAI 兼容端点与 LangSmith Gateway。注册表按精确 model id 为 lead / helper / look / tool selector / learning 五种角色设置 effort、tool search 与输出上限。每次调用按作业、helper、角色计量，LangSmith 全程 trace。

### Skills 布局

- **运行时 skills**（`skills/<name>/SKILL.md`，必须恰好一层）：共 35 个——内容类（`brand-design`、`content-brief`、`stills`、`thumbnails`、`slideshow`、`data-viz`、`design-wiki`、`paper-design`）、视频类（`launch-film`、`faceless-explainer`、`pr-to-video`、`music-to-video`、`general-video`、`storyboard`、`motion-direction`、`motion-graphics`、`embedded-captions`、`gif-edit`、`film-breakdown`、`reference-study`、`remotion-to-hyperframes`）、HyperFrames 系列（`hyperframes` 及 core/animation/audio/cli/creative/keyframes/registry）、通用（`image-generation`、`image-ops`、`sound`、`media-production`、`media-use`、`visual-verify`、`langsmith-trace`）。上游 skill（HyperFrames、`langsmith-trace`）原样 vendored，由 `skills-lock.json` 记录 repo/commit/license 与逐文件 sha256，CI 检查漂移，**不改 vendored 文件**，只在同级 skill 里覆盖。
- **二进制放 `skill-binaries/`**：Context Hub 只收 UTF-8 文本，stager 部署时合并回去。
- **`.agents/skills/`**（给你的编码 Agent 用，不进沙箱）：`design-agent-onboarding / deploy / customize / evals / troubleshoot / update`、`brand-onboarding`、`vendor-sync`、`release-check`，以及 LangChain 的 `deep-agents-core / memory / orchestration`、`managed-deep-agents`、`langsmith-trace / dataset / evaluator`。`AGENTS.md` 把「部署到 Slack」「从官网和品牌指南建 brand pack」「教它做每周 newsletter 头图」「这次改动变好了吗」路由到对应 skill。

### 上手与定制

```bash
curl -fsSL https://raw.githubusercontent.com/langchain-ai/design-agent/main/install.sh | bash
cd design-agent
uv run design-agent config set LANGSMITH_API_KEY=… ANTHROPIC_API_KEY=…
uv run design-agent sandbox publish   # 每个 workspace 一次：Agent 的电脑
uv run mda deploy
```

- 本地零密钥演示：`uv run design-agent sandbox build && uv run design-agent demo`（需 Docker，脚本化模型，产出三种尺寸的发布卡 + 4 秒预告）；有模型 key 后用 `uv run design-agent ask "…"`，结果在 `workspace/out/<thread>/`。
- 定制入口：`workspace/brand/`（品牌）、`OPENAI_API_KEY`（生图）、`ELEVENLABS_API_KEY`（语音/音乐/音效）、`PAPER_MODE=local|bridged`（Paper 画布）、`config/mcp.toml`（Notion/Linear/GitHub/Figma 等 MCP）。授权字体放被 Git 忽略的 `private-fonts/`。
- 成本：代码免费；付模型费用；MDA 托管收费（目前为 LangSmith Cloud 公测）。不需要 Slack 也能用（API / Studio / CLI），但后台长任务（影片、并行 looks）的回报依赖 Slack。
- 仓库纪律：公司私有品牌、字体、参考片、运营历史不得入库；LangChain 品牌与「Content Studio」名称只出现在 README 与 `docs/assets/readme/` 及默认 Slack app 名中，代码与示例保持品牌中立。

## 要点与启示

1. **「代码可数的交给代码」**：颜色、字体、对比度、尺寸、图表数学、结构一致性全部机器校验，把主观的「像不像 AI」转为可验证的契约，是它宣称「不像 AI 做的」的核心。
2. **harness 即产品**：真正的复杂度不在 prompt，而在事件总线、唤醒回执、心跳 ledger、可回放 transcript、memoized 流水线——长任务 Agent 的工程化模式，与 [[mastra]] 的工作流/HITL、[[hermes-agent]] 的闭环学习属同一问题域。
3. **Skills 是知识与流程的载体，但不是权限**：一边用 35 个 skill 承载工作流，一边让能力由代码授予，这个边界是对 [[agent-skills]] 范式的重要补充。
4. **面向团队而非个人**：Slack 线程 + 团队/个人两层记忆 + 每人以自己身份连接 MCP，使它更像「共享设计师」，而非单人 coding agent 的外挂。
5. **视频作为代码**：整条视频链建立在 [[hyperframes]] 的「HTML + 可 seek 时间线 → 确定性 MP4」之上，使分镜、animatic、制作可以由多个 helper 并行、可审查地完成。

## 相关链接

- 仓库：[langchain-ai/design-agent](https://github.com/langchain-ai/design-agent)；博客：[How we built it](https://www.langchain.com/blog/design-agent)
- 视频引擎：[[hyperframes]]（HeyGen 的 HTML→确定性 MP4 框架，Design Agent 所有视频技能的底座）
- 同类「把编码 Agent 变设计引擎」：[[open-design]]（本地 BYOK，31 skills + 72 design systems）、[[impeccable]]（单 skill × 23 命令 × 双 register）——相比之下 Design Agent 走「团队部署 + 沙箱 + 代码闸门」路线，不依赖你的本地编码 Agent
- 图类 skill：[[diagram-design]]、[[excalidraw-diagram-generator]]
- 视频生产对照：[[hypit]]（SVML workflow 化的视频生产框架）
- Agent 框架与 harness 对照：[[mastra]]、[[hermes-agent]]、[[pi]]
- 评测对照：[[reviewbench]]（Agent 评测基准）；Design Agent 自带 `design-agent-evals` skill，改动前后跑 evals 对比
- 上下文与成本：[[headroom]]（上下文压缩层）与其 prompt-cache / 结果落文件 / 工具延迟加载的思路相通
- 概念：[[agent-skills]]、[[llm-knowledge-bases]]（Markdown 文件驱动 LLM 行为的哲学）
