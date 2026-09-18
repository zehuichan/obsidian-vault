---
tags: [hypit, svml, video, ai-agents, skills, hyperframes, typescript, video-clone, ffmpeg, open-source]
created: 2026-09-18
source: "raw/2026-09-18-hypit.md"
---

# Hypit

Hypit 是面向 AI 编码 Agent 的开源视频生产框架：把爆款视频克隆成可编辑、可重跑的 workflow（画面、字幕、B-roll、特效全部锚定在词上而不是秒上），再换脸、换词、换产品，一条命令出大量变体。

## 核心概念

- **Clone 整条 workflow，不只是脚本**：丢进一条参考视频，Agent 重建镜头、节奏、字幕和特效；也可以从模板或纯描述从零写 workflow
- **词锚定，不是秒锚定**：字幕、B-roll、动效绑在 Script 的词、Selection、Moment 上；改一句台词，时间轴自己重排
- **SVML（Semantic Video Markup Language）**：视频的「源代码」；`.svml` 定义视频，`.svs` 定义可复用配方，`.svrun` 声明这次要产出什么
- **Script 是 prose-first**：不含时间码、媒体引用、样式或生成参数；流水线上的一切都读 Script，Script 什么都不读
- **一份 workflow，100 个变体**：复用编排和已有素材，只生成会变的部分；可插拔组件（换主播不动字幕）
- **生成模型可选**：字幕、动效、代码渲染画面可以本地编译成片，不必调用生成 API
- **Skill 与可执行程序分离**：`npx skills add hypit-ai/hypit -g` 装生产知识；`@hypit/hypit` 提供 `hypit` CLI；视频项目可以放在任意位置
- **渲染走 HyperFrames**：字幕、卡拉 OK、排行榜、分屏等是前端代码，无头 Chromium 并发捕获后编码为视频

## 详细内容

### 定位

仓库 [hypit-ai/hypit](https://github.com/hypit-ai/hypit)，官网 [hypit.ai](https://hypit.ai)。标语是 *Clone any viral video with AI agents*。剪藏时约 **9.5k stars / 1.2k forks**，npm 包 `@hypit/hypit` 为 **v0.2.5**，TypeScript monorepo，Node 22.15+、pnpm 10.33、TypeScript 5.9。仓库创建于 2026-07-29。

它不是时间线剪辑器，也不是「再包一层文生视频 API」。对比官网说法：Hypit 做整条片子的克隆（脸、台词、B-roll、字幕、特效），一次 run 可出 10/50/100+ 变体，核心软件费用为 $0；模型走 BYOK 或官方推荐的 HypiHub。Agent 侧兼容 Claude Code、Codex 以及任何支持 Skill 的编码 Agent。

三条入口：

1. **Clone 格式** — 丢参考视频，重建镜头、节奏、字幕、特效，换上你的内容
2. **从描述创作** — 告诉 Agent 想要什么，它规划、生成、组装
3. **用现成模板** — 播客、街头采访、对话、视频通话等已跑通的格式

### 安装与 Agent 工作流

```bash
npx skills add hypit-ai/hypit -g
```

不必 clone 仓库。Skill 提供制作知识；Agent 首次使用时检查并协助安装 `hypit` 可执行程序。Skill 与 executable 走独立更新通道（`hypit version --check`）。

典型五步：

1. 在任意空目录或现有项目里打开支持 Skill 的编码 Agent
2. 给出参考视频（本地文件或支持的平台链接）和要改什么：主播、产品、语言、画幅、CTA
3. Agent 用 WhisperX 做词级对齐，解释片子为什么有效，再选择本地 / 托管服务（HypiHub 或自带 API）
4. 付费生成前先讲清账户、范围、估价；同意后再 `build`。执行与产物落在 Build Results
5. 看成片，可打开 Studio 改时间线；继续对话出变体时，能复用的素材不会重生成

示例提示：

```text
/hypit Clone this video: /path/to/video.mp4, and replace the ranking content
with a comparison of Hypit (hypit.ai) with other AI video products.

/hypit Make a ranking video that puts Hypit in S tier.
```

Agent 环境与模型服务是两件分开的事。OpenAgents 是 Agent 入口合作方；Skill 源码在仓库 `skills/hypit/`。远程 Agent 要注意：远端机器上的 localhost 预览地址不是你电脑上的预览地址。

### SVML 与 Script

Script 是整条片子的语义轴。最小形态：

```svml
<import from="@hypit/script@1"/>

<script id="story">
  <opening>
    <HOST> Hello world.
  </opening>
</script>
```

关键语法：

| 构造 | 作用 |
| --- | --- |
| **Segment** | 有序口播块；标签名即 id，不可嵌套；空 Segment 有结构但不暗示静音时长 |
| **Role Cue** | 标明谁在说（`<ALICE>` / `<BOB>`），不是角色实体，也不选音色；一段里要么全有要么全无 |
| **Dual Text** | `<SVML \| semantic video markup language>`：屏幕显示左边，口播走右边；可为 N:M 对齐 |
| **Cue Break `\|\|`** | 字幕 cue 边界，写在完整对齐单元之间 |
| **Selection `@{id}` / `@{/id}`** | 语义区间（不是帧区间），可交叉，供 B-roll 等组件绑定 |
| **Moment `@{id!}`** | 语义时间点，例如排行榜揭晓瞬间 |

下游用名字引用，而不是秒数：`{story.segment.opening.dialogue}`、`{story.selection.problem}`、`{story.moment.ranking}`。CaptionDocument 含 Display Word、Alignment Unit、Cue Break，仍然没有秒或帧；时间来自后续 Timeline 对齐。汉字/假名按字切分，英文按词切分。

### 包、模型与运行时

视频「要什么」和「谁来做」分开：

| 部分 | 职责 | 选择位置 |
| --- | --- | --- |
| Author component | 把作者输入变成媒体请求或视觉行为 | Source import |
| Model | 定义一次生成请求及其输出 | Source import |
| Provider Endpoint | 经 API 或本地工具执行请求 | Runtime Profile |
| Credential store | 解析 Endpoint 凭据 | Runtime Profile |
| Result repository | 保存 Build 记录与产物 | 项目 Result 配置 |
| Distribution | 官方可执行程序与官方包 | 已安装的 `@hypit/hypit` |

组件通常留在拥有它的视频项目 `packages/` 里；要跨项目再用，可发成版本化 tarball 或 npm 包。官方 Distribution 收录另走 issue。对外扩展针对 `@hypit/hypit/author-kit`、`composition`、`model-kit`、`endpoint-kit` 等公开子路径开发。

公开包导出包括 `narrative`、`temporal`、`spatial`、`timeline`、`svs`、`hyperframes`、`caption`、`speech`、`generation` 等。本地服务有 WhisperX、OpenCV、yt-dlp。内置 Provider 包括 HypiHub、HiAPI、Monid、Pollo、TokenDance，以及本地 media / HyperFrames / WhisperX。

### 渲染与示例成本

字幕、卡拉 OK 高亮、Tier 榜单、评论贴纸、分屏访谈都是前端代码，经 [[hyperframes]] 用无头 Chromium 捕获。三个官方样片都写着 **64 个无头 Chromium 进程并发渲染**。

| 样片 | 时长 / 形态 | 生成栈（摘要） | 官方总成本 |
| --- | --- | --- | --- |
| GOAT DEBATE（UGC 排行） | 20s 足球 Tier List | Seedance 2 Mini 720p A-roll、GPT Image 2 B-roll、WhisperX、音效同步排行榜 | **$1.15** |
| DAILY CREATINE（播客） | 18s 对峙切片 | 三段 A-roll + 生活方式 B-roll、分屏、说话人卡拉 OK 字幕 | **$1.07** |
| NICE RIDE（街头采访） | 26s 三规则揭示 | 人脸框驱动说话人彩色字幕（Google Video Intelligence + YOLOv8 AnimeFace） | **$1.09** |

每个样片附三个 clone：换主播、换主题/产品、换语言或画幅，结构不变。足球样片的 Build 流程示例：`hypit check` → `hypit measure` → `hypit plan` → 确认估价后 `hypit build --follow`。

### 适用产出

信息流广告（从 Meta 广告库克隆，当天出 50 个 Hook；创意衰退后只换开头重跑）、TikTok/Reels/Shorts 爆款模板、带货（同一格式每天换 SKU）、AI UGC 口播、播客/采访切片、纯代码渲染视频、多语言版本。

### 开发仓库

硬依赖只有 Node.js 与 pnpm；Python/uv、ffmpeg、Chrome 只在本地 live Build 时需要。`hypit runtime up` 会准备 Profile 的 programs 并拉 Chromium；`hypit doctor` 只检查不安装。

```text
hypit/
├── packages/     workspace 包
├── docs/         VitePress 文档
├── examples/     可运行示例
├── services/     本地媒体与转写服务
└── test/         仓库边界测试
```

`pnpm check` 做 `tsc --noEmit`；`pnpm test` 走 Node test runner。文档站点：`pnpm docs:dev`。

### 许可

**Hypit Open Source License**（修改版 Apache 2.0）。自用、给客户做片子、单租户自托管可以商用；把 Hypit 做成面向第三方的多租户 / SaaS，或把 Hypit 及其衍生作品作为产品收费分发，需要商业许可。Fork 并按同一许可证开源可以，但不能拿去卖。CLI / run report / manifest 上的名称与 LOGO 不可去掉。你用 Hypit 做出的视频和其他产物归你；第三方模型服务另有条款。贡献者需接受协议日后可收紧或放宽，以及贡献代码可用于官方商业云。

## 相关链接

- [[hyperframes]] — Hypit 本地渲染层：HTML/可 seek 动画 → 无头 Chrome → 确定性 MP4；Hypit 把同一路线接到 SVML 编译图上
- [[agent-skills]] — `/hypit` 按 Skill 规范分发；生产知识与 CLI 分通道更新
- [[opencut]] — 开源时间线剪辑器；Hypit 则把视频写成可重跑的 markup，而不是交互式时间线
- [[pascal-editor]] — 同类「创作工具 + Agent skills」路线，介质是 3D 建筑场景
- [[opencode]] / [[grok-build]] — 可安装 Hypit Skill 的编码 Agent 运行时
- [[vercel-ai-gateway]] — 统一模型网关对照；Hypit 用 Provider Endpoint + HypiHub/HiAPI/Monid/Pollo 做同类解耦

## 参考

- GitHub: https://github.com/hypit-ai/hypit
- 官网: https://hypit.ai
- Quickstart: https://hypit.ai/quickstart
- 开发指南: https://hypit.ai/guide/develop/
- 中文 README: https://github.com/hypit-ai/hypit/blob/main/README.zh-CN.md
- Discord: https://discord.gg/85hnyQnxpn
- Telegram: https://t.me/hypitai
- 原始剪藏: `raw/2026-09-18-hypit.md`
