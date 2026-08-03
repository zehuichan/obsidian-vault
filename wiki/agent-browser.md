---

## tags: [ai-agent, browser-automation, cli, rust, mcp, vercel, open-source, chrome, cdp]
created: 2026-07-27
source: "[https://github.com/vercel-labs/agent-browser](https://github.com/vercel-labs/agent-browser)"

# Agent Browser (`agent-browser`)

Vercel Labs 开源的**面向 AI Agent 的浏览器自动化 CLI**：原生 Rust 实现，通过 accessibility snapshot + `@eN` refs 驱动页面交互，并提供 MCP server、bundled skills、会话/认证持久化与云浏览器 provider。

> 官方定位：Browser automation CLI for AI agents · Fast native Rust CLI · 源码 [vercel-labs/agent-browser](https://github.com/vercel-labs/agent-browser) · Apache-2.0 · ~39k stars（剪藏时）

## 核心概念

- **为 Agent 设计的交互模型**：`snapshot` 输出带 refs 的无障碍树；Agent 用 `@e2` 这类句柄点击/填写，而不是手写脆弱 CSS（传统选择器与语义 locator 仍可用）
- **Client–Daemon 架构**：Rust CLI 解析命令，Rust Daemon 直接走 CDP；daemon 在首次命令时拉起并在命令间常驻，默认约 1 小时空闲后清理（headed / 用户附着浏览器默认不杀）
- **默认 Chrome for Testing**：`agent-browser install` 下载官方自动化频道 Chrome；可检测本机 Chrome / Brave / Playwright / Puppeteer；daemon 本身不依赖 Playwright 或 Node
- **多接入面**：CLI、`agent-browser mcp`（stdio MCP）、`npx skills add vercel-labs/agent-browser` skill、内置 `chat` 自然语言控制
- **安全默认 opt-in**：domain allowlist、content boundary、action policy/confirm、auth vault（加密凭证）、output 长度限制等，启用前不影响现有工作流



## 安装与上手

```bash
npm install -g agent-browser   # 或 brew / cargo install
agent-browser install          # 首次下载 Chrome for Testing
# Linux 系统依赖: agent-browser install --with-deps
agent-browser upgrade          # 按 npm / Homebrew / Cargo 自动升级
```

源码构建需要 Node.js 24+、pnpm 11+、Rust。原生二进制覆盖 macOS/Linux ARM64+x64、Windows x64。

推荐 Agent 工作流：

```bash
agent-browser open example.com
agent-browser snapshot -i --json   # 拿交互元素 + refs
agent-browser click @e2
agent-browser fill @e3 "text"
agent-browser snapshot -i --json   # 页面变化后重新 snapshot
agent-browser screenshot page.png
agent-browser close
```

多步可用 `batch`（参数或 stdin JSON）避免反复起进程；也可用 `&&` 链式调用（daemon 会保持浏览器存活）。

## 能力面（摘要）


| 类别         | 要点                                                                              |
| ---------- | ------------------------------------------------------------------------------- |
| 导航与交互      | open/read/click/fill/type/press/hover/scroll/drag/upload、PDF/截图（含 `--annotate`） |
| Agent 友好读页 | `read`：可不启 Chrome 抓 markdown / `llms.txt` / 可读文本；或读当前 tab 渲染 DOM                |
| 定位         | CSS、`@eN` refs、`find role/text/label/...` 语义 locator                            |
| 会话与认证      | `--session`、`--profile`、`--state`、`--restore`、auth vault、Chrome profile 复用      |
| 网络与调试      | route/mock/HAR、console/errors、trace/profiler、diff snapshot/screenshot/url       |
| 框架特化       | React DevTools tree/inspect/renders/suspense、Web Vitals、`pushstate` SPA 导航      |
| 无障碍        | 内嵌 axe-core 的 `a11y` 审计（需 CDP；Safari/iOS WebDriver 不可用）                         |
| 扩展         | Plugin 协议（credential / browser.provider / launch.mutate / command.run）          |
| 预览         | WebSocket viewport streaming，可人机「pair browse」                                   |


引擎：默认 Chrome（CDP）；`--engine` 可选 `lightpanda`；Safari/iOS 经 WebDriver（`-p ios`，依赖 Appium + XCUITest）。

云浏览器 provider（`-p`）：Browserless、Browserbase、Browser Use、Kernel、AWS Bedrock AgentCore。

## MCP 与 Skills

```bash
agent-browser mcp                    # 默认 tools profile: core
agent-browser mcp --tools all        # 接近完整 CLI 面
agent-browser mcp --tools core,network,react
```

MCP 工具带类型字段与 `allowedDomains` 等，便于客户端展示审批提示。Profiles：`core` / `network` / `state` / `debug` / `tabs` / `react` / `mobile` / `all`。

Skills 与 CLI 版本绑定，避免缓存过期：

```bash
npx skills add vercel-labs/agent-browser
agent-browser skills get core          # 运行时拉取与当前 CLI 匹配的工作流说明
```

兼容 Claude Code、Codex、Cursor、Gemini CLI、Copilot、Goose、OpenCode、Windsurf 等。

## 与本知识库的关系

- 与 [[agent-skills]]：官方推荐 `npx skills add`；stub skill 再 `skills get` 加载与 CLI 同步的正文
- 与 [[opencode]]、[[grok-build]]、[[pi]]、[[hermes-agent]]：编码 Agent 可把本 CLI 当浏览器工具层；OpenCode 等在 skill 安装列表中被明确提及
- 与 [[codegraph]]、[[headroom]]：可分别补「代码结构」与「工具输出压缩」；agent-browser 的 snapshot/MCP 输出本身也可用 Headroom 一类压缩层控制上下文
- 与 [[cua]]：浏览器层 vs 桌面/OS 层——agent-browser 走 CDP + refs；Cua Drivers 做后台原生桌面控制，Sandbox/Bench/Lume 覆盖完整 computer-use 栈



## 风险与局限

- **状态文件含明文 token**（除非设 `AGENT_BROWSER_ENCRYPTION_KEY`）；需进 `.gitignore` 并控制生命周期
- `--remote-debugging-port` **/** `--auto-connect` 把完整浏览器控制暴露给本机进程；仅信任环境使用
- `--allowed-domains` **有前置条件**：不能与某些 profile/restore/state replay / 直接附着浏览器组合；启用时会禁 WebRTC 等旁路
- **iOS 路径重**：需 Xcode/Appium；首启约 30–60s；a11y 审计不支持 Safari/iOS WebDriver
- **点击遮挡**：目标被 banner/modal 盖住时会早失败，需先处理遮挡再重新 snapshot



## 相关链接

- 仓库：[github.com/vercel-labs/agent-browser](https://github.com/vercel-labs/agent-browser)
- Skill 目录：[skills.sh/vercel-labs/agent-browser](https://skills.sh/vercel-labs/agent-browser)
- 原始剪藏：`raw/2026-07-27-agent-browser.md`
- 同类对照：[[agent-skills]]、[[opencode]]、[[grok-build]]、[[pi]]、[[hermes-agent]]、[[cua]]

