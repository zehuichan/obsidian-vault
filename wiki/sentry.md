---
tags: [sentry, error-tracking, apm, devops, monitoring, python, django, open-source]
created: 2026-07-06
source: "raw/2026-07-06-sentry-error-tracking.md"
---

# Sentry

Sentry 是一个面向开发者的调试平台，用于检测、追踪和修复生产环境中的错误与性能问题；`getsentry/sentry` 仓库是其核心服务端与 Web UI 的开源实现。

## 核心概念

- **错误追踪（Error Tracking）**：自动捕获未处理异常、堆栈、上下文（用户、设备、release），把「日志里的线索」变成可操作的 issue
- **性能监控（APM）**：追踪 transaction、span，定位慢请求与瓶颈，与错误数据在同一平台关联分析
- **开发者优先**：SDK 覆盖 20+ 语言/框架，集成通常只需几行初始化代码
- **Monorepo 架构**：后端 Python（Django）+ 前端 TypeScript/React，前后端分离部署
- **Fair Source 许可**：核心代码开源，商业 SaaS 与 self-hosted 并行
- **SDK 与服务端分离**：本仓库是 platform/server；各语言 SDK 在独立仓库（如 `sentry-javascript`、`sentry-python`）

## 详细内容

### 产品定位

Sentry 的口号是 *"Users and logs provide clues. Sentry provides answers."* —— 传统日志告诉你「发生了什么」，Sentry 进一步提供：

1. **结构化 issue 聚合**：相同 root cause 的异常自动合并，减少告警噪音
2. **完整堆栈与 breadcrumbs**：复现路径、用户操作序列、release 变更
3. **分布式追踪**：跨服务 trace，把前端、API、后台任务串成一条链路
4. **Release 健康度**：新版本上线后错误率、crash-free session 等指标

Topics 涵盖 `error-monitoring`、`crash-reporting`、`apm`、`csp-report`、`devops` 等，定位是生产可观测性里的「应用层调试平台」，而非纯基础设施监控（如 Prometheus 指标采集）。

### 仓库架构

`getsentry/sentry` 是 Sentry 平台本体，语言构成约 **Python 57.9% + TypeScript 40.9%**：

| 目录 | 职责 |
|------|------|
| `src/` | 后端 Python：Django API、业务逻辑、数据处理 |
| `static/` | 前端 React/TypeScript：Sentry Web UI |
| `tests/` | 后端 pytest 测试 |
| `self-hosted/` | 自托管部署配置 |
| `api-docs/` | API 文档 |
| `devservices/` | 本地开发依赖服务（Kafka、Redis 等） |

开发模式上，前后端**非原子部署**——改 API 与改 UI 需拆成独立 PR，后端先行。本地开发通过 `devenv sync` + `devservices up` + `devservices serve`（全栈）或 `pnpm run dev-ui`（仅前端，API 代理到生产）启动。

Feature flag 体系称 FlagPole：新功能在 `src/sentry/features/temporary.py` 注册，Python 侧 `features.has(...)`，前端 `organization.features.includes(...)` 检查。

### SDK 生态

官方 SDK 覆盖主流运行时，各 SDK 独立仓库维护：

- **Web / 移动端**：JavaScript、Electron、React Native、Dart/Flutter
- **后端**：Python、Ruby、PHP、Laravel、Go、Rust、Java/Kotlin、Elixir、Clojure、Perl
- **原生 / 游戏**：C/C++、Objective-C/Swift、C#/F#、Unity、Unreal、Godot
- **其他**：PowerShell

集成模式统一：初始化 SDK → 自动捕获未处理异常 → 上报到 Sentry ingest endpoint → 平台聚合、告警、展示。

### 部署选项

1. **Sentry SaaS**（[sentry.io](https://sentry.io)）：托管服务，按事件量计费
2. **Self-hosted**：`self-hosted/` 目录提供 Docker Compose 等方案，适合内网或合规要求
3. **开发/贡献**：Fair Source 许可下可 fork 本仓库参与贡献（44k+ stars，活跃社区）

最新 release 26.6.0（2026-06-15），仓库累计 10 万+ commits。

### 与 DevOps / 前端生态的关系

Sentry 位于应用可观测性层，与 [[verdaccio]] 这类包管理基础设施、[[codemirror]] / [[ilha]] 这类前端库无直接耦合，但常被同一工程团队采用：

- CI/CD 中按 release 标记部署版本，Sentry 按 release 对比错误率
- 前端 SPA 通过 `@sentry/browser` 捕获 JS 错误与 performance
- 后端 Django/FastAPI 等服务通过 Python SDK 上报

在 AI Agent 工作流场景中，Sentry 也可监控 Agent 运行时异常（如 tool call 失败、LLM API 超时），与 [[codegraph]]、[[openspec]] 等开发工具链互补——前者管「运行时出了什么问题」，后者管「代码结构与规格怎么组织」。

## 相关链接

- 仓库：[getsentry/sentry](https://github.com/getsentry/sentry)
- 文档：[docs.sentry.io](https://docs.sentry.io/)
- 讨论：[GitHub Discussions](https://github.com/getsentry/sentry/discussions)
- 同主题基础设施：[[verdaccio]]
- 代码理解 / Agent 工具：[[codegraph]]、[[agent-skills]]
