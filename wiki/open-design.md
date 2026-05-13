---
tags: [llm, design, agent, skills, byok, open-source, design-system]
created: 2026-05-05
source: "https://open-design.ai/"
---

# Open Design

由 nexu-io 在柏林发起的开源设计工作室与工具集，定位为 "Claude Design 的开源替代"。它**不自带 LLM**，而是把你已有的编码 Agent（Claude、Codex、Cursor、Gemini、OpenCode、Qwen 等）转化为设计引擎，由 31 个可组合的 `SKILL.md` 与 72 个品牌级 `DESIGN.md` 系统驱动。当前版本 v0.3.0，Apache-2.0 许可。

## 核心理念

> "We treat your agent as a creative collaborator, not a black box."
>
> "The strongest coding agents already live on your laptop. We don't ship one — we wire them into a skill-driven design workflow."

不发布新模型，而是**把已有 Agent 接入一个由文件驱动的设计工作流**：本地运行（`pnpm tools-dev`），Web 层部署到 Vercel，每一层都坚持 BYOK（Bring Your Own Key）。

这与 [[llm-knowledge-bases]] 共享同一种底层哲学——**让结构化的 Markdown 文件驱动 LLM 行为，而非依赖封闭产品**。Open Design 把这套思路用在视觉创作领域，Karpathy 的 LLM Wiki 则用在知识管理领域。

## 四面一环（4 Surfaces / 1 Loop）

### Skills, not plugins

31 个文件式 `SKILL.md` 包。把文件夹丢进项目，重启 daemon，技能即可加载。**技能就是文件**——不是插件 API、不是私有运行时。

### Design Systems as Markdown

72 个可移植的 `DESIGN.md` 设计系统：Linear、Vercel、Stripe、Apple、Cursor、Figma 等品牌的视觉语言被编码为 Markdown，可被任意 Agent 理解和复用。

### 12 Agent Adapters

支持 12 种主流 CLI Agent，自动扫描 `$PATH` 检测：
Claude · Codex · Gemini · Cursor · Copilot · OpenCode · Devin · Hermes · Pi · Kimi · Kiro · Qwen

### BYOK at Every Layer

OpenAI 兼容代理。DeepSeek、Groq、OpenRouter、自托管 vLLM——粘贴 baseUrl + key 即可工作。

## 工作流：Detect → Discover → Direct → Deliver

四阶段循环，每阶段都是迭代的、可视的、研究驱动的——由可组合的文件构成，而非不透明的 prompt。

### 1. Detect（探测）

Daemon 启动时扫描 `$PATH` 上的 12 个编码 Agent，自动加载 31 个 skills + 72 个 systems。

### 2. Discover（发现）

第一轮交互不是自由对话，而是一份**结构化问题表单**：surface（界面类型）、audience（受众）、tone（语气）、scale（规模）、brand context（品牌语境）。30 秒锁定方向。

> "30s of radios beats 30min of redirects."（30 秒的单选题胜过 30 分钟的来回纠偏。）

### 3. Direct（指挥）

从 5 个**确定性视觉方向**中选一个：OKLch 色彩空间的调色板、字体栈、布局姿态线索（layout posture cues）。

### 4. Deliver（交付）

Agent 写入磁盘 → 用户在沙箱化的 iframe 中预览 → 导出 HTML / PDF / PPTX / ZIP / Markdown。

> "Skills inform everything. Files make it real."

## 五个进行中的实验

来自 Labs / Skills Catalog（5 of 31 ongoing）：

1. **Magazine Decks**（Deck）：用 `guizang-ppt` 做编辑级幻灯片，杂志排版、WebGL 主视觉。
2. **Synthetic Matter**（Media）：Gpt-image-2 + Seedance + HyperFrames。图像、视频、音频与代码共享同一聊天界面。
3. **Prompt Choreography**（Loop）：在改进任何一个像素之前，先弹出交互式问题表单。
4. **Visual Reasoning**（Critique）：5 维自我批评门控每个产物——philosophy · hierarchy · execution · specificity · restraint（哲学 · 层级 · 执行 · 特异性 · 克制）。
5. **Soft Systems**（Runtime）：沙箱化 iframe 预览、流式 todos、真实 cwd 文件系统、人机自适应循环。

## 关键作品

- **guizang-ppt**（Featured skill 01/31）：杂志式 Web PPT，面向产品发布和路演。
- **kami**（Companion system 04/72）：编辑式纸张系统——温暖的羊皮纸画布、墨水蓝点缀、衬线主导的层级，多语言原生设计（EN · zh-CN · ja）。

## 设计哲学：5 维自我批评

Open Design 把视觉评审编码进了 critique skill：

- **Philosophy（哲学）**：作品要表达什么立场？
- **Hierarchy（层级）**：信息架构是否清晰？
- **Execution（执行）**：细节打磨是否到位？
- **Specificity（特异性）**：是否避免了通用模板感？
- **Restraint（克制）**：是否做到了减法？

每一件 LLM 产出的视觉物件都需要通过这 5 维自评闸门。

## 谱系（Lineage）

Open Design 自述站在以下开源团队的肩膀上：

- **huashu-design** — Philosophy
- **guizang-ppt** — Decks
- **open-codesign** — UX
- **Devin CLI** — Terminal
- **hyperframes** — Frames

主要贡献者来自 23 个城市，6 名核心维护者，包括 @tw93（kami）、@op7418（guizang）、@alchaincyf（huashu）等。

## 启示

1. **Skill-as-File 是 Agent 时代的可移植性单元**：与 [[llm-knowledge-bases]] 中的 wiki 一样，Open Design 把"能力"也降维到 Markdown 文件——意味着任何兼容的 Agent 都能加载。
2. **结构化问询替代自由对话**：用 30 秒的表单代替反复的 prompt 调整，把"提问"本身变成一个可编排的环节。
3. **设计系统作为知识资产**：72 个 `DESIGN.md` 系统让 Linear、Vercel、Stripe 等品牌的视觉语言变得可被 LLM 直接调用。
4. **BYOK + 本地优先**：避免被某一个模型供应商锁定，反映了 2026 年开源生态对"主权"的重视。

## 相关链接

- 官网：[open-design.ai](https://open-design.ai/)
- GitHub：[nexu-io/open-design](https://github.com/nexu-io/open-design)
- 相关概念：[[llm-knowledge-bases]]、[[agent-skills]]
- 同路异轨：[[impeccable]] — 同样把 AI 编码助手转为设计引擎，但走"1 skill × 23 命令 × 双 register"的纵向路线，而非 Open Design 的"31 skills + 72 design systems"横向铺开
