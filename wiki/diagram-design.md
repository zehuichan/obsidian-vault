---
tags: [agent-skills, diagram, design-system, svg, html, mermaid, drawio, ai-harness, open-source]
created: 2026-09-08
source: "raw/2026-09-08-diagram-design.md"
---

# Diagram Design

Cathryn Lavery 开源的 **Agent Skill**：让 Claude Code、Codex、Factory Droid、Pi 等兼容 Agent Skills 的宿主，生成**编辑级**（editorial）自包含 HTML + SVG 图——无阴影、无圆角盒子 slop、也不是把 Mermaid 原样渲染出来。截至剪藏（2026-09-08）仓库约 **33.9k stars / 2.2k forks**，MIT；正文宣称 **39** 种视觉类型（GitHub 简介仍写 38，以 README 为准）。

> *The highest-quality move is usually deletion.* 每个节点都要挣到位置。强调色只留给读者该先看的 1–2 处。目标密度：4/10。

## 核心概念

- **编辑级图，不是工程草图**：输出是可双击打开的单文件 HTML（内联 SVG），默认无 JS、无构建、无外链图片；三种皮肤：minimal light / minimal dark / full-editorial
- **行为与布局解耦**：先选 semantic pattern（队列、策略对比、信任边界等），再落到最近的视觉类型；pattern 不新增类型计数
- **品牌 60 秒 onboard**：从网站抽取 paper / ink / muted / accent / 字体，写入 `references/style-guide.md`；多客户用 `~/.diagram-design/profiles/` + 项目 `.diagram-design` marker
- **重绘，不是转换**：draw.io / Mermaid 被解析成中间表示后，按 format × size × detail × audience 四档旋钮重新排进本设计系统，并给出 fidelity ledger
- **渐进披露 Skill**：启动只见 name + description；常规请求只加载 `SKILL.md` + 一份 `type-*.md`；语义、动画、导入规范按需再读——教科书级的 [[agent-skills]] L1/L2/L3

## 为什么存在

作者在 littlemight.com 写稿、兼做 BestSelf.co。向 Claude 要架构图 / 流程图 / 金字塔时，得到的是和站点完全不像的通用圆角盒；要么在 Figma 耗半小时，要么干脆不画。于是做成 Skill：39 种类型、读一次网站就能贴合品牌。

默认调色：白烟纸面、jet-black 墨、atomic-tangerine 强调、blue-slate 次要、银色发丝线。4px 网格（所有坐标/宽/间距必须整除 4）被当成「看起来不像 AI 生成」的硬约束。

## 39 种视觉类型

每种三套静态变体。v2.5.10 补齐最后十种（Sankey、鱼骨、Wardley、看板、用户旅程、部署、依赖图、UML class、story map、数据库 schema）。现场画廊：[cathrynlavery.github.io/diagram-design](https://cathrynlavery.github.io/diagram-design/)

| 类型 | 用途 | 类型 | 用途 |
|------|------|------|------|
| Architecture | 组件 + 连接 | IT current-state | 遗产景观 + 现代化 |
| Flowchart | 决策逻辑 | Sequence | 时序消息 |
| State machine | 状态 + 转移 | ER / data model | 实体 + 字段 |
| Timeline | 轴上事件 | Swimlane | 跨职能流 |
| Quadrant | 双轴定位 | Radar / spider | 多轴对比 |
| Loop / flywheel | 强化循环 + 共享枢纽 | Nested | 用包含表达层级 |
| Tree | 父子 | Org chart | 所有权 + 路由 |
| Layer stack | 堆叠抽象 | Venn | 集合重叠 |
| Pyramid / funnel | 排序层级或漏斗 | Bar / Line / Scatter | 分类、趋势、分布 |
| Treemap | 面积表达部分–整体 | Gantt | 任务 + 阶段 |
| High-Level | 集群上的端到端栈 | Process | 多角色顺序工作流 |
| Medallion | 多层数据存储 | Data flow | 按角色的管道步骤 |
| DP integration | 源 → 核心 → 消费 | DP security matrix | 按角色权限 |
| Sankey | 分流/汇合的量 | Fishbone | 分组原因 → 一果 |
| Wardley map | 价值链 × 演化 | Kanban | 按状态的在制品 |
| User journey | 阶段、动作、情绪 | Deployment | 区、主机、制品 |
| Dependency graph | 扇入、秩、环 | UML class | 类、操作、类型化关系 |
| Story map | 骨干 × 发布切片 | Database schema | 物理表 + 列级 FK |
| Polar chart | 周期幅值 · 线性半径 | | |

另有三个 **primitive**（不是新类型）：annotation callout（斜体衬线 + 虚线贝塞尔指引）、sketchy 滤镜（手绘，适合随笔不适合技术文档）、87 个单色 IT/云图标（Tabler MIT + Simple Icons CC0，`currentColor` 继承皮肤）。

可选 **motion** 也不增加类型：`none`（默认、无脚本）/ `reveal` / `step` / `loop`；`prefers-reduced-motion` 只展示完整静态首帧。控制器必须与审过的 `template-motion.html` 字节一致，禁止远程资源、CSS `@import`、可执行 HTML 属性。

八个 routed **semantic patterns** 覆盖：扇入队列与瓶颈、重复阶段槽、非结构化输入变换、成对策略轨迹、secure paved road、治理目录、补偿性安全层、可追溯块分解。

## 安装

**Claude Code：**

```text
/plugin marketplace add cathrynlavery/diagram-design
/plugin install diagram-design@diagram-design
```

第三方 marketplace 默认关自动更新，需在 `/plugin` → Marketplaces → diagram-design 打开 **Enable auto-update**。

**Codex：** `codex plugin marketplace add cathrynlavery/diagram-design` 然后 `codex plugin add diagram-design@diagram-design`。

**Factory Droid：** `droid plugin marketplace add` 该 GitHub URL，再 `droid plugin install diagram-design@diagram-design --scope user`（按 commit 跟踪，不是 manifest 显示版本）。

**Pi：** `pi install https://github.com/cathrynlavery/diagram-design`，会话里 `/reload`；显式调用 `/skill:diagram-design`。Pi 无自动刷新，更新用 `pi update --extensions`。

**Kiro / OpenCode / Cursor 等：** 共用 `skills/diagram-design/`。Kiro 从子目录 URL 导入；OpenCode 需拷贝或 symlink 到 `.opencode/skills/`（无 marketplace 包）。可编辑安装应 clone 后 symlink，以免托管更新覆盖改过的 `style-guide.md`；已保存的 profile 与带 marker 的项目不受影响。

旧的 `npx skills add` 独立副本**不会**自动跟上 Codex marketplace，需卸掉再按各宿主原生路径重装。

## 品牌 Onboarding

```
You:     "onboard diagram-design to https://yoursite.com"
Agent:   抓首页 → 抽主色与字体 → 映射到 paper/ink/muted/accent/link → 出示 diff → 写入 style-guide.md
You:     "yes, apply it"
```

| 站点检测到的 | 变成 |
|--------------|------|
| `<body>` 背景 | `paper` |
| 主文本色 | `ink` |
| 次要/说明文字 | `muted` |
| 卡片/容器 | `paper-2` |
| CTA / 链接 / 标题主色 | `accent` |
| `<h1>` 字体 | title |
| body 字体 | node-name |
| code/pre 字体 | sublabel |

写入前做 WCAG AA（`ink` over `paper`）；图内字号 9–12px 过不了对比度会提出调整值。SVG 默认 `role="img"` + `aria-labelledby` + `<title>`/`<desc>`，ID 按图与变体加前缀，方便一页内联多张。公开站点字体直接引用并在渲染后校验，而不是偷偷换成系统字体。

**First-run gate：** 新项目若 `style-guide.md` 仍是默认，第一次出图会停下来问：走 onboarding、手贴 token，还是继续用默认——避免把默认皮肤静默塞进已有品牌项目。

多客户：onboard 一次存成 named profile，项目放 `.diagram-design`（`profile: <slug>`），工作区读 `~/.diagram-design/profiles/<slug>.md`，不互相覆盖共享安装里的 style-guide。

## 导入 draw.io / Mermaid，导出 PNG / SVG

导入是**按目的地重绘**：

| 旋钮 | 选项 | 改什么 |
|------|------|--------|
| Format | html · svg · png · html+png | 交付物 |
| Size | doc-inline / doc-wide / slide-16x9 / slide-4x3 / social-* / print-* / fit | viewBox **和字号阶梯**（投影幻灯片用 16px 节点名，不是 12px） |
| Detail | faithful ≤24 · balanced ≤12 · simplified ≤7 | 固定降级阶梯：装饰 → 重复 → 叶簇 → 基础设施 |
| Audience | engineer · mixed · executive | **措辞**不是节点数：`Auth / JWT·RS256·:8443` → `token check` → `Sign-in` |

draw.io 读 `.drawio` / `.drawio.xml` / `.drawio.png`（内嵌）/ `.drawio.svg`（含压缩 payload）。Mermaid 读 `.mmd` / `.mermaid` / Markdown 围栏块。只解析文本：不渲染、不跑 JS、不联网、不跟点击目标。

**永不带走：** 源坐标、源调色、源字体、draw.io 斜线意大利面、Mermaid 自动布局。**永远带走：** 组件、关系、分组、方向。每次导入附 fidelity ledger（合并/折叠/丢弃了什么）。

导出：斜杠命令 `/diagram-design:export-diagram`（Pi 为 `/export-diagram`）。SVG 抽出 `<svg>` 并注入 Google Fonts；PNG 默认 Playwright 2×。`--registry` 给可追溯块分解 pattern 再吐一份 `*.registry.json`。动效 HTML 导出前要走到显式终态（`?motion=static`）。

## Skill 架构

`SKILL.md` 先路由行为再路由布局。仓库同时带 Claude / Codex / Factory marketplace manifest、`commands/`（插件斜杠命令）和 `prompts/`（Pi 模板）。共享技能在 `skills/diagram-design/`。

| 你说… | Agent 加载 |
|--------|------------|
| 「给我一张流程图」 | `SKILL.md` + `type-flowchart.md` |
| 「对比这两次策略请求为何不同」 | + `semantic-patterns.md` |
| 「给这段策略轨迹做动画」 | 再加 `animation.md` |
| 「onboard 到我的站点」 | `onboarding.md` + `style-guide.md` |
| 「重绘这个 .drawio / Mermaid」 | 对应 import 参考 + `output-spec.md` + 选中的 type |

类型再多，一次也只读需要的那一份。质量门很重：`lint-skin.py`（源）+ `lint-render.py`（无头 Chromium 用像素差分测裁切，而不是 `getBoundingClientRect`）、几何标签遮挡、treemap 面积编码、Sankey 守恒、block registry 树、docs/路由面同步、装好后的 `self_check.py`。CI 跑 Linux / Windows / macOS。

**判定它在工作：** 常规请求只多加载一份 type 参考；画之前先声明类型、pattern、尺寸、计划砍掉什么；输出是一个离线可开的 `.html`（除 Google Fonts 无网络）；读屏能报出标题和描述；`self_check.py` 打印 `OK`；onboard 后有 fidelity receipt。

## 设计系统（一段话）

一种强调色，每图 1–2 个焦点。三套字体：Instrument Serif（标题与斜体旁注）、Geist sans（节点名）、Geist Mono（端口、URL、字段类型——不是整图「开发者风」）。1px 发丝边、无阴影、圆角 ≤10px。坐标/宽/间距必须被 4 整除。珊瑚色焦点节点把视线拉到真正重要的 1–2 处。完整规范在 `SKILL.md`。

## 何时不要用

- 推文/终端里的快速 unicode 图 → 用 wiretext 类 skill
- 任何清单 → 表格或项目符号
- 前后对比 → 表格
- 单框加标签 → 写成一句话

画之前问：读者从这张图学到的，会不会比一段写好的话更多？不会就别画。

## 与既有文章的关系

- [[agent-skills]] — 几乎是规范文档里「渐进加载 Generator」的完整落地：description 触发、`SKILL.md` 路由、`references/` 按类型拆、脚本做抽取与自检
- [[impeccable]] — 同属「把设计素养装进编码 Agent」；Impeccable 管产品/营销 UI 与反 slop 命令，Diagram Design 只管**图**，用 4px 网格、单强调色和删除密度对抗圆角盒 slop
- [[open-design]] — Open Design 横向铺 31 skills + 72 `DESIGN.md`；这里是单 skill + 一份 `style-guide.md` + 网站抽取 token
- [[excalidraw-diagram-generator]] / [[excalidraw]] — 前者输出手绘风 `.excalidraw` JSON（九类图）；Diagram Design 输出编辑级 HTML/SVG（39 类），并强调「不要 Mermaid slop」。草图协作仍用 Excalidraw；要刊用级架构/时序/Wardley 用本 skill
- [[pi]] / [[opencode]] — README 把二者列为宿主：Pi 用 `pi install` + prompt 模板；OpenCode 无 marketplace，靠拷贝/symlink `skills/diagram-design/`

## 相关链接

- 仓库：[github.com/cathrynlavery/diagram-design](https://github.com/cathrynlavery/diagram-design)
- 画廊：[cathrynlavery.github.io/diagram-design](https://cathrynlavery.github.io/diagram-design/)
- 作者：[littlemight.com](https://littlemight.com) · [BestSelf.co](https://bestself.co) · [X @cathrynlavery](https://x.com/cathrynlavery)
- 原始剪藏：`raw/2026-09-08-diagram-design.md`
- 规范与同类：[[agent-skills]]、[[impeccable]]、[[open-design]]、[[excalidraw-diagram-generator]]、[[pi]]
