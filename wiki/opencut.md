---
tags: [opencut, video-editor, open-source, capcut, rust, typescript, mcp, plugins]
created: 2026-07-20
source: "raw/2026-07-20-opencut.md"
---

# OpenCut

OpenCut 是一个免费开源的视频编辑器（自称 CapCut 替代品），目标覆盖 Web、桌面与移动端；当前主仓库正在从零重写，生产站仍跑 classic 版本。

## 核心概念

- **开源 CapCut 替代**：MIT 许可，面向创作者的免费剪辑工具，官网 [opencut.app](https://opencut.app)
- **双轨演进**：classic（可用）与 rewrite（架构中）；日常使用走 classic，新架构预览在 [new.opencut.app](https://new.opencut.app)
- **Plugin-first**：重写以插件为一等公民，配套 Editor API，便于第三方扩展
- **Rust 核心 + 多端**：桌面 / 移动 / 浏览器共用一套 Rust core，上层以 TypeScript 为主
- **Agent 友好**：计划内置 MCP server、headless 批渲染，以及编辑器内 scripting tab
- **暂不开放外部贡献**：架构设计阶段只欢迎 Discord / Issue 讨论，尚未接受外部 PR

## 详细内容

### 产品定位

OpenCut 定位是「自由开源的跨端视频编辑器」。GitHub 仓库 [OpenCut-app/OpenCut](https://github.com/OpenCut-app/OpenCut) 标语为 *The open-source CapCut alternative*；语言构成约 **TypeScript 97% + CSS + 少量 Rust**（Rust 占比会随重写推进上升）。

Topics：`editor`、`oss`、`videoeditor`。

### Classic vs Rewrite

| 维度 | Classic | Rewrite（当前 main） |
|------|---------|----------------------|
| 仓库 | [opencut-classic](https://github.com/opencut-app/opencut-classic) | [OpenCut-app/OpenCut](https://github.com/OpenCut-app/OpenCut) |
| 线上入口 | [opencut.app](https://opencut.app) | [new.opencut.app](https://new.opencut.app)（预览） |
| 建议用途 | **今天能用的版本** | 跟进新架构，尚未接班 |
| 贡献 | 视 classic 仓库策略 | 架构未定型，暂不接外部贡献 |

重写路线图要点：

1. **Editor API** — 程序化操控时间线 / 素材 / 导出
2. **第三方插件** — plugin-first 架构
3. **统一 codebase** — Rust core 驱动 desktop / mobile / browser
4. **MCP server** — 让 AI Agent 直接操作剪辑能力（与仓库内 [[codegraph]]、[[hermes-agent]] 等 Agent 工具生态可对照）
5. **Headless mode** — 自动化、批量渲染
6. **Scripting tab** — 编辑器内脚本环境

### 本地开发

工具链基于 [proto](https://moonrepo.dev/proto) + [moon](https://moonrepo.dev/moon)：

```sh
# 安装 proto 后，在仓库根目录
proto use                 # 按 .prototools 安装固定工具链

moon run web:dev          # localhost:5173
moon run api:dev          # localhost:8787
moon run desktop:dev      # 见 apps/desktop/README.md
```

仓库顶层可见 `apps/`、`.moon/`、`Cargo.toml`、`bunfig.toml` 等，典型 monorepo：Bun + moon 任务 + Rust workspace。

### 生态与赞助

- 社区：Discord、X (@opencutapp)
- 赞助商示例：fal.ai（生成式图像 / 视频 / 音频模型）
- 许可：MIT

## 相关链接

- [[codegraph]] — 本地代码图 MCP，可与 OpenCut 计划中的 MCP server 对照「Agent 如何接工具」
- [[hermes-agent]] — 开源 AI Agent，适合想象「Agent + 视频编辑 MCP」的使用场景
- [[ilha]] — 同类「小而可嵌入」的前端架构参考（同为现代 TS 开源库生态）
