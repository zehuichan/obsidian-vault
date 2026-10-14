---
tags: [e2e, testing, ai-agent, playwright, mobile, typescript, open-source]
created: 2026-10-14
source: "raw/2026-10-14-e2e-testing-framework.md"
---

# e2e（TesterArmy）

e2e 是 TesterArmy 开源的 **Web 与移动端端到端测试框架**：同一条测试里可以写精确 locator，也可以用自然语言让 agent 完成目标，再用断言核对结果。

> 官方定位：Next generation e2e testing framework for web and mobile apps · 仓库 [tester-army/e2e](https://github.com/tester-army/e2e) · 文档 [e2e.tester.army/docs](https://e2e.tester.army/docs) · Apache-2.0 · 剪藏时约 10.3k stars / 497 forks · 通向 1.0，小版本间 API 与配置仍可能变

没有 agent 步骤的测试不需要模型。自带订阅、API key 或本地模型。`e2e` 包把全部文档打进 `node_modules/e2e/docs`，编码 Agent 可离线读。

## 核心概念

- **三种步骤混用**：`agent.act` 完成一个目标（有缓存时可回放、不调模型）；`agent.assert` / `agent.waitFor` / `agent.extract` 判断或抽取屏幕（内置 agent 下始终调模型）；`screen` + `expect` 是精确 locator，不调模型
- **回放缓存**：一次 `act` 只有在后续检查通过后才写入记录；下次从同一路由重放动作，控件或终态对不上时把控制权交回 agent
- **引擎即 target**：Web 经 Playwright 驱动 Chromium / Firefox / WebKit；Mobile 经 agent-device 驱动 iOS 模拟器与 Android 模拟器；也可自写引擎
- **模型自备**：配置里没有默认模型。内置 agent 需要支持 tool call 与图像的 AI SDK 模型；Clef、Jev 一类 decision model 走自定义 executor
- **Runner 与模型分工**：executor 选动作并给出 verdict；runner 负责脱敏、授权并执行动作、预算、填入 secret、写报告

```ts
import { test, expect } from 'e2e';

test('a member upgrades to Pro', async ({ app, agent, screen }) => {
  await app.open('/settings/billing');

  await agent.act('upgrade the workspace to the Pro plan');
  await agent.assert('the invoice preview shows a prorated amount');

  await expect(screen.getByRole('status')).toContainText('Pro');
});
```

## 详细内容

### 上手

需要 Node.js 24.8+（Node 22 则 22.22.3+）。Windows 上要在 WSL 里跑。移动测试还要 Xcode 与 iOS simulator runtime，或带模拟器的 Android SDK。

```bash
npx e2e init          # 或 pnpm dlx / bunx；--yes 走非交互默认
npx e2e run
npx e2e run tests/signup.e2e.ts:12
npx e2e run --grep checkout --target web
npx e2e run --headed      # 看浏览器
npx e2e run --no-cache    # 跳过回放
```

`init` 询问引擎（Web 或 Mobile）和模型提供商（也可选 None），再写入 `e2e.config.ts`、示例测试、依赖，并可装 coding-agent skill 与 MCP。已有文件会保留。`npx e2e init --yes` 写出一份用 Vercel AI Gateway 的 Web 配置，此时还不装依赖。`npx e2e guide` 打印 skill，`npx e2e guide <topic>` 打印单个主题。

示例 Web 配置（向导按所选 provider 改写）：

```ts
import type { E2EConfig } from 'e2e';
import { web } from '@e2e-dev/web';
import { gateway } from 'ai';

export default {
  agents: { default: { model: gateway('openai/gpt-6-luna-fast') } },
  targets: [{ engine: web(), app: { url: 'http://localhost:3000' } }],
} satisfies E2EConfig;
```

iOS 示例把 engine 换成 `mobile({ platform: 'ios' })`，`app` 用 `bundleId`（如 `Settings`），并设 `workers: 1`。Android 则 `platform: 'android'`、`bundleId: 'com.android.settings'`。先跑 `npx agent-device doctor`。`app.open()` 会全新启动应用；不调用则从上一条测试留下的设备状态继续。

仓库 [`examples/`](https://github.com/tester-army/e2e/tree/main/examples) 有可独立运行且已通过的套件：Vite、Next.js、Astro、Expo、SwiftUI、Jetpack Compose、Kotlin Multiplatform、Flutter。

把下面这种 prompt 交给 Claude Code、Codex、Cursor 等，agent 会装 skill，之后「给 checkout 加一条测试」不必再做安装：

1. `npx e2e init --yes`（或本项目的 pnpm / bun 等价命令）
2. 读 skill，并按它完成后续步骤
3. 把 target 指到本应用：Web 设 URL 和 dev server 命令；iOS/Android 换成 `@e2e-dev/mobile` 并改写示例
4. 选定模型：ChatGPT、GitHub Copilot、OpenCode Console、SuperGrok 订阅，或 API key / 本地模型；需要登录时跑对应的 `npx e2e login` 并等人确认
5. 装依赖、跑通示例
6. 为最重要的用户流程写一条测试并迭代到通过

### 目标、断言、locator

每个 agent 调用、screen 调用和 `expect.poll` 都要 `await`。测试在步骤仍在跑时返回，会以 `STEP_NOT_AWAITED` 失败。

`agent.act` 一次只给一个目标，完成则返回，失败则抛出。顺序由测试作者控制。屏幕上的用词写进指令；测试数据放 `params`。密码用 `Secret`，模型看不到值。每次运行都变的值用 `unique()`，回放时代入当前值：

```ts
import { unique } from 'e2e';

const email = `ada+${Date.now()}@example.test`;
await agent.act('sign up with email {email}', { params: { email: unique(email) } });
```

`unique()` 用于流程会填入或核对的数据；会改变流程走向的选择（例如套餐名）用普通 param。某个 `unique()` 值等于或包含在另一个 param 里时，该步不记录（`param-collision`）。

断言看的是**当前屏幕**，看不到先前步骤，也看不到 `act` 的摘要：

| API | 行为 |
| --- | --- |
| `agent.assert` | 立刻判断。陈述为假 → `ASSERTION_FAILED`；证据不足 → `ASSERTION_INCONCLUSIVE` |
| `agent.waitFor` | 等到条件为真；屏幕不变时不调模型 |
| `agent.extract` | 按 Standard Schema（如 Zod）从屏幕抽出结构化数据；数据不在屏上则 `ASSERTION_INCONCLUSIVE`，不返回占位值 |

图表、布局等视觉判断加 `{ vision: true }`；只看截图用 `vision: 'only'`。截图增加输入 token。secret 填入之后，该次尝试不再向模型发送截图。模型生成的文案核对语义；精确文案用 locator。可另设 `judge` 模型，只负责 assert / waitFor / extract。

Locator 在已知控件和精确值时使用，优先级是 role + name，其次 label，最后 test id。查询命中多个节点时，动作、读取和单节点断言会失败，用 `.filter()`、`.first()`、`.nth()` 收窄。`expect` 会重试到通过或超时；`textContent()` 这类直接读取等到匹配后只读一次。从 `@e2e-dev/web` 或 `@e2e-dev/mobile` 导入 `test`，才能给 `browser` / `device` fixture 带上类型。`device.locator` 使用 agent-device 选择器。

组织方式与常见 runner 相近：`describe` 可带 `tags`；`beforeEach` 准备数据；`test.skip(condition, reason)`；`{ serial: true }` 让一组测试在同一 worker 上按序跑并一起重试；`test.extend` 做自定义 fixture。每条测试有自己的 attempt。

### Agent 步骤看见什么、能做什么

内置 agent 是默认 executor。`act` 的首轮 prompt 含指令、params、已完成步骤，以及脱敏后的文本快照（role、name、文本、状态）。不含原始 HTML、cookie、header 或环境变量。密码字段被遮住，屏幕上出现的已登记 secret 变成占位。动作之后模型收到屏幕差异；变化很大时才给新的完整快照。节点在元素还在时保持同一 id。

判断类步骤只看到指令、当前屏幕和 agent 的 `context`。

工具由引擎提供，例如 `tap`、`type`、`press`、`select`、`scroll`、`navigate`。文本快照描述不了控件时，可用像素工具截图并按坐标操作。带 `Secret` param 时多一个 `type_secret`：模型只看到名字和用途，runner 核对字段后自己填值。值不进入 prompt、transcript、报告或 AI trace。单次定向动作最多等 15 秒。项目工具可声明预算和平台。

循环会发现重复调用。连续三次动作失败会要求换做法；第五次要求给出 verdict。预算或截止时间临近时也会要求 verdict。

Verdict 有三种：`passed`；`failed`（条件为假或无法证明，证据不足是 `ASSERTION_INCONCLUSIVE`）；`blocked`（凭证、环境、种子数据或预算在产品判断之前拦住了步骤）。只有后两种带错误码。凭证无效或未配置、种子数据缺失、测试准备失败退出码为 2；环境不可用为 3；预算耗尽、超时、自动化不支持为 1。

`assert` 与 `extract` 做一次判断，答案格式坏了再修一次。`act` 只在文本快照不够时才要截图。回放缓存让已通过的 `act` 不再调模型。

### 回放缓存

本地默认 `read-write`（回放并记录）。CI 且未设置 `cache` 时默认 `read-only`（只回放）。`--no-cache` 或 `cache: 'off'` 两者都不做。自定义 `cache.store` 在 CI 里不会自动变成 `read-only`。`e2e explore` 和 `e2e mcp` 不用缓存。

记录写入时机：`read-write` 下，attempt 结束时，若该 `act` 之后有检查通过，才保存。算作验证的包括 locator matcher、引擎断言（如 `toHaveURL`）、`locator.waitFor()`、`agent.assert`、`agent.waitFor`。不算的包括普通值的 `expect`、`expect.poll`、`textContent()` 等读取、`agent.extract`、另一次 `act`，以及 attempt 仅仅通过。失败的 attempt 仍保留失败点之前已验证的步骤，并删掉未验证条目和卷入失败的回放。取消运行不改缓存。`MODEL_UNAVAILABLE` 与 `MODEL_PROVIDER_FAILED`（缺 key、限流、5xx、120 秒无响应）不驱逐记录，因为它们说明不了应用是否变了。

一条缓存属于一个测试、一个 target、一条指令、一组 params 和一个 agent。改名、改指令、改普通 param、换 agent 或改 `context` / `agentContext`、换引擎、改 project id 或应用身份都会 miss。换模型不会 miss。升级引擎本身也不会；新版本解析出不同节点时，在回放阶段 miss。预览 URL 每次部署都变时，设 `app.identity` 保持同一缓存身份。设备上回放用屏幕标题当路径。

回放四步：起始屏幕必须是同一路由；按 role、name、test id 和周围上下文找到控件并重复动作（缺失则等待，匹配含糊则停止）；核对最终路由以及出现/消失的控件及其 checked、selected、expanded、pressed 状态；然后无模型结束，或把已跑内容和停止原因交给 agent 续跑。Secret 按名字解析当前值并重新授权。

路由比较 origin、path、query。记录 id 和 fragment 不算。path 或 query 里的 id、token、时间戳、cache buster 当成 id 占位，所以 `/orders/42?t=…` 与 `/orders/7?t=…` 是同一路由。`?mode=safe` 与 `?mode=unsafe`、`/companies` 与 `/companies?tab=notes` 则不同。应用自身 origin 上的地址按 path 比较，因此同一 `app.identity` 的预览部署可以沿用记录；其他 origin 是另一条路由。

运行摘要把结果分成 `replayed`（`self-finalized`）、`handed off`（`agent-concluded`）、`missed`。常见原因：`no-entry`、`retry`（重试默认不回放，除非 `--strict-cache`）、`invalid-entry`、`truncated`、`wrong-context`、`target-not-found`（15 秒内没出现）、`target-ambiguous`、`gap`、`action-failed`、`action-uncertain`、`viewport-changed`、`end-mismatch`。`gap` 包括会改状态的项目工具，以及模型从屏幕、截图或日期推出来的值。记录在超过 50 个动作、键入超过 4096 字符、键入内容含已登记 secret，或控件没有可用描述时被截断。

条目是 `.e2e/cache/` 下按 key 哈希命名的 JSON：动作、目标描述、键入文本、步骤摘要，以及终态检查（路由，加上最多 8 个出现和 8 个消失的控件）。不含 prompt、对话、截图、观察树或 secret 值。其他键入内容按原文保存，分享前要审阅。`init` 把该目录写入 `.gitignore`。要在 CI 复用就删掉这行并提交；CI 仍是 `read-only`，除非显式 `cache: 'read-write'`。

`--strict-cache`（或 `cache.strict`）让过期记录以 `REPLAY_STALE` 失败（退出码 2，不重试、不调模型）。从未记录过的新步骤、`gap`、`truncated` 仍走实时。strict 运行即使在 `read-write` 下也不写、不删条目，所以新步骤要在一次非 strict 的运行里录下来再提交。默认不 strict 时，过期记录只是把步骤交回 agent，之后每次 CI 都付模型费用，直到有人重新录制。

```bash
npx e2e cache ls
npx e2e cache stats
npx e2e cache clear
```

报告里的 `cached` 百分比是供应商的 prompt cache，和这条回放缓存不是一回事。

### 模型与订阅

内置 agent 依赖 `ai` 包，模型必须是 AI SDK 实例，不能是字符串。没有默认模型。Quickstart 示例走 `gateway()`，凭 `AI_GATEWAY_API_KEY`，或在没有该 key 时用 Vercel OIDC（`vercel login` + `vercel link`）。也可换 OpenRouter、各家 provider 包，或 `@ai-sdk/openai-compatible` 指到本地端点（如 `http://127.0.0.1:11434/v1`）。e2e 不自动加载 `.env`，要在配置里 `process.loadEnvFile()` 或由 shell 导出。

| 计划 | 连接方式 |
| --- | --- |
| ChatGPT Plus / Pro | `npx e2e login openai` |
| GitHub Copilot | `npx e2e login github-copilot`（已有 GitHub CLI 登录则可复用） |
| OpenCode Console（Zen / Go） | `npx e2e login opencode-console` |
| SuperGrok 或 X Premium+ | `npx e2e login spacexai` |
| Claude Max / Team | 使用该计划每月 API credits 的 API key |

DeepSeek V4 与 Meta Muse Spark 拒绝 agent 步骤要求的强制 tool choice。e2e 会改用 `auto` 并指示只以 tool call 回复，该模型上的后续步骤沿用这一方式。

对 OpenAI / Azure Responses 模型，请求带 `store: false` 和 prompt cache key，提供商不保留对话副本。要保留则在 `providerOptions` 里打开 `store`。Anthropic 的 BAA 只覆盖直连 `api.anthropic.com`（`@ai-sdk/anthropic`），不要经 gateway。

多个模型做成多个具名 agent，用 `--agent` 或测试选项挑选。`--debug` 可看每步输入 token 里供应商 prompt cache 的占比。

### 包与托管运行时

| 包 | 作用 |
| --- | --- |
| `e2e` | SDK、runner、CLI |
| `@e2e-dev/web` | Playwright：Chromium、Firefox、WebKit |
| `@e2e-dev/mobile` | agent-device：iOS / Android 模拟器与模拟器 |
| `@e2e-dev/github` | 把结果发成 pull request 评论 |
| `@e2e-dev/kernel` | Kernel 托管浏览器 |
| `@e2e-dev/limrun` | Limrun 托管 iOS 模拟器与 Android 模拟器 |
| `@e2e-dev/eas` | EAS Simulators |
| `@e2e-dev/smol` | 本机温热 Chromium microVM；默认每次尝试从中分出一支 |
| `@e2e-dev/decision` | 有界语义动作与断言的 decision-model executor |

文档还覆盖 API 测试（与 UI 测试同一 runner）、视觉对比、登录会话复用、bug bash（并行 explore 再用失败测试证明 bug）、persona、GitHub Actions / EAS Workflows / Bitrise / Codemagic，以及从 Playwright、Cypress、Selenium、Detox、Maestro 迁移。

### 遥测与安全

CLI 发送匿名用量：哪些命令和引擎在跑、失败发生在哪里。不发送测试内容、应用内容或凭证。关闭：`npx e2e telemetry disable` 或 `E2E_TELEMETRY_DISABLED=1`。字段列表在[遥测文档](https://e2e.tester.army/docs/telemetry)。漏洞不要开公开 issue，寄到 security@tester.army。

## 与本知识库的关系

- 与 [[agent-browser]]：agent-browser 是给编码 Agent 用的浏览器 CLI（snapshot refs / CDP）；e2e 是测试 runner，Web 引擎走 Playwright，失败与产物进报告，验证过的 `act` 可以回放
- 与 [[cua]]：Cua 提供桌面 computer-use 驱动、沙箱和评测环境；e2e 把自然语言目标写进可重复的 Web/移动测试，而不是后台桌面控制
- 与 [[vercel-ai-gateway]]：quickstart 与示例配置默认 `gateway()`；也可直连 provider 或本地 OpenAI-compatible 端点
- 与 [[opencode]]、[[grok-build]]：订阅登录分别对应 `npx e2e login opencode-console` 与 `npx e2e login spacexai`；编码 Agent 也可按 quickstart prompt 在应用目录里装 skill 并写测试
- 与 [[agent-skills]]：`init` 可写入 e2e skill；`npx e2e guide` 打印与当前 CLI 一致的说明
- 与 [[standard-json]]：`agent.extract` 的 schema 是 Standard Schema（文档示例用 Zod）

## 相关链接

- 仓库：[github.com/tester-army/e2e](https://github.com/tester-army/e2e)
- 文档：[e2e.tester.army/docs](https://e2e.tester.army/docs) · [核心概念](https://e2e.tester.army/docs/core-concepts) · [回放缓存](https://e2e.tester.army/docs/cache) · [模型](https://e2e.tester.army/docs/models)
- 产品站：[tester.army/e2e](https://tester.army/e2e)
- npm：[`e2e`](https://www.npmjs.com/package/e2e)
- 原始剪藏：`raw/2026-10-14-e2e-testing-framework.md`
- 同类对照：[[agent-browser]]、[[cua]]、[[vercel-ai-gateway]]、[[opencode]]、[[grok-build]]、[[agent-skills]]
