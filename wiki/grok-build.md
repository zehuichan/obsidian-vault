---
tags: [ai-agent, coding-agent, terminal, tui, rust, spacexai, grok, mcp, acp, open-source]
created: 2026-07-25
source: "https://github.com/xai-org/grok-build"
---

# Grok Build (`grok`)

SpaceXAI 开源的**终端 AI 编码 Agent harness + TUI**：全屏、可鼠标交互、可扩展；能理解代码库、改文件、跑 shell、搜网页、管长任务；支持交互式、无头（脚本/CI），以及通过 Agent Client Protocol（ACP）嵌入编辑器。

> 官方定位：SpaceXAI's coding agent harness and TUI · 产品页 [x.ai/cli](https://x.ai/cli) · 源码 [xai-org/grok-build](https://github.com/xai-org/grok-build)

## 核心概念

- **Rust 实现的 CLI/TUI**：仓库是 SpaceXAI monorepo 的周期性同步树；根目录 `SOURCE_REV` 记录对应 monorepo commit SHA
- **三种运行形态**：交互 TUI · headless（脚本/CI）· ACP 嵌入编辑器
- **完整 Agent 栈**：工具（终端/编辑/搜索等）+ workspace（文件系统/VCS/执行/checkpoint）+ MCP / skills / plugins / hooks / sandbox
- **官方二进制名 `grok`**：源码构建产物叫 `xai-grok-pager`，安装脚本装成 `grok`
- **外部分支不接受贡献**（见 `CONTRIBUTING.md`）；首方代码 Apache-2.0

## 安装与上手

预编译包支持 macOS / Linux / Windows：

```sh
curl -fsSL https://x.ai/cli/install.sh | bash   # macOS / Linux / Git Bash
irm https://x.ai/cli/install.ps1 | iex          # Windows PowerShell
grok --version
```

从源码构建需要：Rust（`rust-toolchain.toml` 钉死版本）、DotSlash（`bin/protoc` 等 hermetic 工具）、protoc。macOS/Linux 为正式构建主机；Windows 为 best-effort。

```sh
cargo run -p xai-grok-pager-bin              # 构建并启动 TUI
cargo build -p xai-grok-pager-bin --release  # release: target/release/xai-grok-pager
```

首次启动会打开浏览器做认证。变更日志：[x.ai/build/changelog](https://x.ai/build/changelog)。

## 仓库结构（摘要）

| Crate / 路径 | 职责 |
|--------------|------|
| `xai-grok-pager-bin` | 组合根，产出二进制 |
| `xai-grok-pager` | TUI（scrollback、prompt、modal、渲染）与用户指南 |
| `xai-grok-shell` | Agent runtime + leader/stdio/headless 入口 |
| `xai-grok-tools` | 工具实现（terminal、file edit、search…） |
| `xai-grok-workspace` | 主机文件系统、VCS、执行、checkpoints |
| 其余 `crates/codegen/...` | config、MCP、markdown、sandbox 等闭合依赖 |

根 `Cargo.toml` 是**生成物**，只读；应改各 crate 自己的 `Cargo.toml`。开发时用 `cargo check/test/clippy -p <crate>`，避免整 workspace 全量构建。

文档入口：[docs.x.ai/build/overview](https://docs.x.ai/build/overview)；用户指南在 `crates/codegen/xai-grok-pager/docs/user-guide/`（快捷键、slash 命令、配置、主题、MCP、skills、plugins、hooks、headless、sandbox 等）。

## 与本知识库的关系

- 与 [[opencode]]：同属终端编码 Agent；OpenCode 是独立开源产品（多端 + 多 provider）；Grok Build 是 SpaceXAI 自家 harness，且 `THIRD-PARTY-NOTICES` 标明工具层含 **sst/opencode**（及 openai/codex）的 in-tree port
- 与 [[pi]]：都是「harness + TUI」路线；pi 刻意极简、扩展驱动，Grok Build 功能面更全（MCP、hooks、sandbox、ACP、checkpoint 等）
- 与 [[hermes-agent]]、[[trellis]]：同属 Agent 运行时/脚手架生态；Hermes 偏自我改进与多平台网关，Trellis 偏仓库内 wiki/workflow 注入多 Agent
- 与 [[agent-skills]]：用户指南明确支持 skills；可按 skill 规范叠加可移植行为模块
- 与 [[codegraph]]、[[headroom]]：可挂在周边补「结构理解」与「上下文压缩」

## 风险与局限

- **源码树是 monorepo 同步快照**：公开仓库未必与内部最新一致；以 `SOURCE_REV` 与发布 changelog 为准
- **不接受外部 PR**：想贡献只能 fork 自用，不能指望上游合入
- **Windows 源码构建未正式支持**：日常用预编译安装脚本更稳
- **第三方 port 许可需单独核对**：工具实现含 codex / opencode 等移植，细节见仓库 `THIRD-PARTY-NOTICES`

## 相关链接

- 仓库：[github.com/xai-org/grok-build](https://github.com/xai-org/grok-build)
- 产品页：[x.ai/cli](https://x.ai/cli)
- 文档：[docs.x.ai/build/overview](https://docs.x.ai/build/overview)
- 原始剪藏：`raw/2026-07-25-grok-build.md`
- 同类对照：[[opencode]]、[[pi]]、[[hermes-agent]]、[[agent-skills]]、[[trellis]]
