---
tags: [ai-agent, coding-agent, terminal, desktop, ide-extension, multi-provider, lsp, open-source, privacy]
created: 2026-07-21
source: "https://opencode.ai/"
---

# OpenCode

Anomaly（anomalyco）开源的 **AI 编码 Agent**：可在终端（TUI）、桌面 App、IDE 扩展中使用；支持 75+ LLM 提供商（含本地模型），可复用 GitHub Copilot / ChatGPT Plus·Pro 订阅，并强调不存储代码与上下文（隐私优先）。官网宣称约 160K GitHub stars、900 贡献者、每月约 7.5M 开发者使用。

> 官方定位：**The open source AI coding agent** · Free models included or connect any model from any provider

## 核心能力

| 能力 | 说明 |
|------|------|
| **LSP enabled** | 自动为 LLM 加载合适的 Language Server |
| **Multi-session** | 同一项目上并行启动多个 Agent |
| **Share links** | 将会话分享为链接，便于协作或排错 |
| **订阅复用** | GitHub 登录用 Copilot；OpenAI 登录用 ChatGPT Plus/Pro |
| **Any model** | 经 Models.dev 接入 75+ 提供商，含本地模型 |
| **Any editor** | 终端界面 + 桌面 App + IDE 扩展 |

桌面端已推出带 **tabs** 的 beta（macOS / Windows / Linux）。

## 安装与上手

安装脚本（推荐）：

```bash
curl -fsSL https://opencode.ai/install | bash
```

其他方式（文档）：`npm install -g opencode-ai`、Homebrew（建议用 `anomalyco/tap/opencode`）、Arch（`pacman` / AUR）、Windows 上 Chocolatey / Scoop / Mise，以及 Docker：`docker run -it --rm ghcr.io/anomalyco/opencode`。

典型流程：

1. `/connect` — 配置提供商（新手可用 **OpenCode Zen** 精选模型）
2. 进入项目目录后运行 `opencode`
3. `/init` — 分析项目并生成根目录 `AGENTS.md`
4. 用 **Tab** 在 **Plan**（只规划、不改代码）与 **Build** 模式间切换
5. `/undo` / `/redo` — 撤销或重做 Agent 改动；`/share` — 分享当前会话

## Zen（精选模型层）

**Zen** 是 OpenCode 团队测试、对标过的 coding-agent 模型清单，目标是减少各提供商质量不一致带来的踩坑。新手文档建议优先通过 `/connect` 选 opencode，再到 [opencode.ai/auth](https://opencode.ai/auth) 完成计费与 API key。

## 隐私立场

官网明确：**不存储你的代码或上下文数据**，以便在隐私敏感环境中使用。这与「把会话上传给第三方做训练/遥测」类产品形成对照——具体边界仍以官方隐私说明为准。

## 与本知识库的关系

- 与 [[pi]]：同属终端编码 Agent + 多 provider + 可复用订阅；pi 刻意极简、扩展驱动，OpenCode 则内置 LSP、Plan/Build、多会话、分享链接、桌面/IDE 形态更「产品化」
- 与 [[hermes-agent]]：都是开源编码 Agent；Hermes 偏自我改进闭环与多平台接入，OpenCode 偏多端入口与提供商生态
- 与 [[trellis]]：Trellis 把仓库内 wiki/workflow 注入多种 Agent；OpenCode 的 `/init` → `AGENTS.md` 是同类「项目级持久指令」入口
- 与 [[agent-skills]]、[[openspec]]：可叠加——用 skill / spec 工作流约束 OpenCode 的 Plan→Build 循环
- 与 [[codegraph]]、[[headroom]]：分别补「结构理解」与「上下文压缩」，可挂在 Agent 周边提升效率

## 风险与局限

- **体量大、迭代快**：社区指标很高，API / 安装通道多，跟进文档比跟传闻更稳妥
- **FAQ 在官网上多为折叠项**：计费、是否必须额外订阅等细节需对照 Docs / Zen 页面核实
- **隐私「不存储」≠ 模型提供商不处理**：请求仍会发往所选 LLM 提供商；合规要按 provider 单独评估

## 相关链接

- 官网：[opencode.ai](https://opencode.ai/)
- 文档：[opencode.ai/docs](https://opencode.ai/docs)
- 原始剪藏：`raw/2026-07-21-opencode.md`
- 同类对照：[[pi]]、[[hermes-agent]]、[[trellis]]、[[openspec]]、[[agent-skills]]
