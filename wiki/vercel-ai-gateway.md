---
tags: [ai-gateway, vercel, llm, ai-sdk, byok, openai-compatible, anthropic]
created: 2026-08-20
source: "https://vercel.com/docs/ai-gateway"
---

# Vercel AI Gateway

Vercel 提供的统一 AI API：一把密钥访问数百个模型（文本 / 图像 / 视频 / 语音 / embeddings），token 按供应商原价计费、零加价，失败时自动换供应商重试。

## 核心概念

- **One key, hundreds of models**：一张 `AI_GATEWAY_API_KEY` 覆盖多家供应商；模型 ID 形如 `creator/model-name`（如 `openai/gpt-5.6-sol`、`anthropic/claude-opus-5`、`xai/grok-4.5`）
- **Unified API**：同一套调用切模型，不必改供应商 SDK；AI SDK 里把 `model` 写成字符串即默认走 Gateway
- **High reliability**：某供应商失败会自动重试其他供应商；可配 provider 过滤、排序、超时、model fallbacks、routing rules
- **No markup**：token 价与直连供应商相同；[BYOK](https://vercel.com/docs/ai-gateway/authentication-and-byok/byok) 同样零加价
- **Embeddings + spend monitoring**：向量检索与跨供应商花费监控
- **兼容面**：AI SDK v5/v6、OpenAI Chat Completions、OpenAI Responses、Anthropic Messages、OpenResponses，以及框架集成

## 详细内容

### 它提供什么

官方定位是「一个 endpoint 接数百个模型」。能力面包括文本生成（含 tool calling / 结构化输出）、图像与视频生成、TTS / STT、Realtime 语音 Agent、embeddings、web search，以及可观测性（用量、延迟、花费）。

Getting Started 按模态拆 quickstart：

| 模态 | 典型模型（文档示例） |
|------|----------------------|
| Text | GPT-5.5、Claude Opus 4.7、Gemini 3.1 Pro、Llama 4 等 100+ |
| Image | Flux 2 Flex、Recraft V3、Imagen 等 |
| Video | Veo 3.1、KlingAI、Wan、Grok Imagine Video 等（单次请求或后台 job） |
| Realtime | WebSocket 低延迟 speech-to-speech |
| Speech | tts-1、Whisper 等 |

模型目录：Dashboard → AI Gateway → Model List，或公开页 [vercel.com/ai-gateway/models](https://vercel.com/ai-gateway/models)，或未鉴权 REST：`GET https://ai-gateway.vercel.sh/v1/models`。AI SDK 可用 `gateway.getAvailableModels()`。

同一模型可能由多家供应商托管，规格（价、延迟、uptime）不同，可在 [leaderboards](https://vercel.com/docs/ai-gateway/leaderboards) 对比。

### 鉴权：API key、OIDC、BYOK

**每次请求都必须用 Vercel 身份**（Gateway API key 或 OIDC）。BYOK 只决定 Gateway *如何去调供应商*，不能替代这次鉴权。

1. Dashboard 的 [AI Gateway API Keys](https://vercel.com/d?to=%2F%5Bteam%5D%2F%7E%2Fai-gateway%2Fapi-keys&title=AI+Gateway+API+Keys) 创建密钥
2. 本地：`AI_GATEWAY_API_KEY=...`（AI SDK 会自动读）
3. 部署在 Vercel 上：可用 `VERCEL_OIDC_TOKEN`，无需长期密钥。文档推荐本地 key、线上 OIDC：

```ts
const apiKey = process.env.AI_GATEWAY_API_KEY || process.env.VERCEL_OIDC_TOKEN;
```

注意：团队成员离队后，其创建的 API key 会被停用。需要与人解绑的鉴权，用 OIDC。

**BYOK**（付费档）：在团队级挂自己的供应商凭证，走企业价/credits、私有能力、零加价。失败时 Gateway 会回退到系统凭证（回退用量记入 credits）。凭证全团队共享。也可在单次请求里用 `providerOptions.gateway.byok` 传入（此时不看 Dashboard 缓存的 BYOK）。Spend 通过自有 key 的部分**不计入 budgets**。

ZDR（Zero Data Retention）开启时，默认跳过 BYOK；只有单独标为 ZDR-compliant 的 key 才会进入 ZDR 路由集。`inferenceRegion` 对 BYOK 同样生效。

### 第一次文本请求（AI SDK）

```bash
pnpm add ai dotenv @types/node tsx typescript
```

```ts
import { streamText } from 'ai';
import 'dotenv/config';

const result = streamText({
  model: 'openai/gpt-5.6-sol',
  prompt: 'Invent a new holiday and describe its traditions.',
});

for await (const textPart of result.textStream) {
  process.stdout.write(textPart);
}
```

把 `model` 写成普通字符串时，AI SDK 自动用 Vercel AI Gateway provider。也可显式 `import { gateway } from '@ai-sdk/gateway'`，或 `createGateway({ apiKey, baseURL })`（默认 base：`https://ai-gateway.vercel.sh/v1/ai`）。

应用级默认供应商可在启动时改：`globalThis.AI_SDK_DEFAULT_PROVIDER = openai`（Next.js 可放 `instrumentation.ts`）。

### 兼容 API（同一把 Gateway key）

Base URL 因协议略有不同：

| 协议 | Base URL | 说明 |
|------|----------|------|
| OpenAI Chat Completions | `https://ai-gateway.vercel.sh/v1` | 任意 OpenAI SDK / HTTP 客户端 |
| Anthropic Messages | `https://ai-gateway.vercel.sh` | 任意 Anthropic SDK |
| OpenResponses | `https://ai-gateway.vercel.sh/v1/responses` | [openresponses.org](https://openresponses.org) 开放标准 |

模型字段仍用 Gateway slug（例如用 OpenAI SDK 调 `anthropic/claude-opus-5`）。这就是「一把 key、换模型几乎不改代码」的落地方式。

### 路由与运维

可配置 provider options、过滤/排序、model fallbacks、超时、routing rules；另有 automatic caching、service tiers、uptime/metrics、reasoning、web search。文档另有 coding agents、disallow prompt training、app attribution。

## 相关链接

- [[mastra]] — TypeScript Agent 框架，自带 model routing；应用侧可与 AI SDK / Gateway 叠用
- [[opencode]] — 开源编码 Agent，多 provider；可用 Gateway 做统一模型入口（Vercel 有 OpenCode + Gateway 的实践文）
- [[agent-browser]] — 同属 Vercel Labs 的 Agent 工具链（浏览器自动化），可与 Gateway 上的 coding agents 对照
- [[grok-build]] — Grok 模型可通过 Gateway 的 `xai/...` slug 调用
- [[headroom]] — 上下文压缩，降低经 Gateway 发出的输入 token

## 参考

- 文档总览：[vercel.com/docs/ai-gateway](https://vercel.com/docs/ai-gateway)
- Getting started：[vercel.com/docs/ai-gateway/getting-started](https://vercel.com/docs/ai-gateway/getting-started)
- 鉴权与 BYOK：[vercel.com/docs/ai-gateway/authentication-and-byok](https://vercel.com/docs/ai-gateway/authentication-and-byok)
- 模型目录：[vercel.com/ai-gateway/models](https://vercel.com/ai-gateway/models) · `GET https://ai-gateway.vercel.sh/v1/models`
- AI SDK provider：[ai-sdk.dev/providers/ai-sdk-providers/ai-gateway](https://ai-sdk.dev/providers/ai-sdk-providers/ai-gateway)
- 原始剪藏：`raw/2026-08-20-vercel-ai-gateway.md`
