---
tags: [video, html, agents, skills, heygen, ffmpeg, puppeteer, animation, open-source]
created: 2026-08-09
source: "https://github.com/heygen-com/hyperframes"
---

# HyperFrames

HeyGen 开源的 HTML 原生视频框架：用 HTML/CSS/媒体 + 可 seek 的动画写成构图，经无头 Chrome 逐帧捕获、FFmpeg 编码为确定性 MP4；面向本地 CLI、AI 编码 Agent skills，以及托管创作流水线。

## 核心概念

- **HTML-native composition**：构图就是带 `data-*` 时序属性的 HTML（`data-start` / `data-duration` / `data-track-index`），无 React、无专有时间线格式、无强制构建步骤
- **Seekable animation adapters**：GSAP、CSS、Lottie、Three.js、Anime.js、WAAPI 或自定义 frame adapter；动画必须可 seek，才能帧精确渲染
- **Deterministic render**：同输入 → 同帧 → 同输出；适合 CI、回归测试与自动化出片
- **Agent skills（19 个）**：`/hyperframes` 作路由；创作工作流（产品发布、无脸讲解、PR→视频等）按需安装；领域 skill 覆盖 core / animation / keyframes / creative / media / CLI / registry / Figma
- **frame.md**：把面向网页的 `design.md` 反转为「为镜头写」的设计系统，供 Agent 按 token/规则组视频而不猜尺度
- **Catalog + Lambda**：可 `hyperframes add` 安装转场/叠加/图表等积木；支持本地、HeyGen cloud、AWS Lambda 分布式渲染

## 详细内容

### 定位与对比 Remotion

与 [Remotion](https://www.remotion.dev) 同属「无头 Chrome + FFmpeg 出片」路线。Remotion 押注 React 组件与 bundler；HyperFrames 押注人类与 Agent 都能直接写的纯 HTML。许可为 Apache 2.0（Remotion 为 source-available）。仓库约 40k+ stars（剪藏时），TypeScript monorepo，HeyGen 生产在用；社区示例含 tldraw、TanStack 等。

### 快速上手

Agent 侧（推荐）：

```bash
npx skills add heygen-com/hyperframes --full-depth
# 非交互 / Agent：用 npx hyperframes skills update 只装 core set
```

CLI 侧：

```bash
npx hyperframes init my-video
cd my-video
npx hyperframes preview   # 浏览器 live reload
npx hyperframes render    # 出 MP4
```

依赖：Node.js 22+、FFmpeg。

### 构图契约（示例）

舞台用 `data-composition-id` / 分辨率；clip 用 `class="clip"` + 时序属性；GSAP timeline 需 `paused: true` 并挂到 `window.__timelines[compositionId]`，供引擎按帧 seek。

### Skills 地图（精简）

| 层 | 代表 skill |
| --- | --- |
| Router | `/hyperframes` — 任何「做/改/渲视频」请求先读它 |
| Creation | `/product-launch-video`、`/faceless-explainer`、`/pr-to-video`、`/embedded-captions`、`/talking-head-recut`、`/motion-graphics`、`/music-to-video`、`/slideshow`、`/general-video`、`/remotion-to-hyperframes` |
| Domain | `/hyperframes-core`、`animation`、`keyframes`、`creative`、`media-use`、`cli`、`registry`、`/figma` |

默认只装 **core set**；创作工作流由路由按需 `skills update` 拉取，避免一次装全 19 个。

### 包结构

| Package | 职责 |
| --- | --- |
| `hyperframes` | CLI：init / lint / preview / render / publish / doctor / cloud / lambda |
| `@hyperframes/core` | 类型、解析、生成、linter、runtime、frame adapters |
| `@hyperframes/engine` | Puppeteer + FFmpeg 的 page→video 捕获引擎 |
| `@hyperframes/producer` | 捕获、编码、混音完整流水线 |
| `@hyperframes/studio` | 浏览器构图编辑 UI（仍在演进） |
| `@hyperframes/player` | 可嵌入播放器 web component |
| `@hyperframes/shader-transitions` | WebGL 着色器转场 |
| `@hyperframes/aws-lambda` | 分布式渲染部署与 SDK |

开发克隆注意：`packages/producer/tests/**/output.mp4` 走 Git LFS（约 240 MB）；只要源码可用 `GIT_LFS_SKIP_SMUDGE=1`。

### 适用产出

产品发布与功能公告、带 code diff 的 PR walkthrough、数据可视化/地图动画、社媒 kinetic captions、docs/PDF/站点导览讲解、自动化内容管线中的可复用 motion graphics。

## 相关链接

- [[agent-skills]] — Skill 规范与渐进加载；HyperFrames 用 router + on-demand creation workflows 实践同一思路
- [[open-design]] — 官方指南含 Open Design handoff；`frame.md` / `DESIGN.md` 与 Open Design 的设计系统 Markdown 哲学相邻
- [[impeccable]] — 设计向 Agent Skill 包；可与 HyperFrames 的 creative / frame.md 层对照
- [[mastra]] — 同为 TypeScript Agent 生态；Mastra 偏应用/工作流，HyperFrames 偏 HTML→视频渲染

## 参考

- GitHub: https://github.com/heygen-com/hyperframes
- Docs: https://hyperframes.heygen.com/introduction
- Quickstart: https://hyperframes.heygen.com/quickstart
- Showcase: https://hyperframes.heygen.com/showcase
- Playground: https://www.hyperframes.dev/
- vs Remotion: https://hyperframes.heygen.com/guides/hyperframes-vs-remotion
- Discord: https://discord.gg/EbK98HBPdk
- 原始剪藏: `raw/2026-08-09-hyperframes.md`
