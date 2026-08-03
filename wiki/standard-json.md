---
tags: [standard-schema, json-schema, zod, valibot, typescript, validation, open-source]
created: 2026-08-03
source: "raw/2026-08-03-standard-json.md"
---

# standard-json

standard-json（`@standard-community/standard-json`）是把实现 [Standard Schema](https://github.com/standard-schema/standard-schema) 接口的校验库 schema 统一转换成 JSON Schema 的转换器（standard-community, MIT）。

## 核心概念

- **Standard Schema 入口**：通过 `schema["~standard"].vendor` 识别厂商，再分发到对应转换实现
- **统一 API**：`toJsonSchema(schema)` 异步转换；配合 `loadVendor` 可同步扩展/覆盖厂商
- **按需 peer 依赖**：核心包轻量（依赖 `quansync`）；Zod v3 / Valibot 等需另装厂商专用转换包
- **多厂商覆盖**：peer 面包含 Zod、Valibot、ArkType、Effect、TypeBox、Sury 等（多为 optional）
- **可插拔转换器**：未内置支持的库可自写 convertor 并 `loadVendor` 注册

## 详细内容

### 解决什么问题

生态里 Zod、Valibot、ArkType 等各自有（或没有）「schema → JSON Schema」路径，API 不统一。standard-json 站在 Standard Schema 规范之上，提供单一 `toJsonSchema`，便于把运行时校验 schema 接到 OpenAPI、表单、MCP tool input、文档生成等需要 JSON Schema 的场景。

### 安装

```sh
pnpm add @standard-community/standard-json
```

部分厂商需额外安装转换依赖：

| Vendor | Package |
|--------|---------|
| Zod v3 | `zod-to-json-schema` |
| Valibot | `@valibot/to-json-schema` |

### 基本用法

```ts
import { toJsonSchema } from "@standard-community/standard-json";

const jsonSchema = await toJsonSchema(schema);
```

### 扩展与定制（sync / loadVendor）

`loadVendor(vendorName, convertor)` 有两类用途：

1. **为未支持的校验库补转换**：库需实现 Standard Schema；用 `schema["~standard"].vendor` 作为 vendor 名注册。
2. **覆盖已支持厂商的转换函数**：例如用 `@standard-community/standard-json/zod` 的 handler，或传入自定义实现。

注册后可按同步方式调用 `toJsonSchema`（取决于是否异步加载厂商实现）。

### 包形态（约 v0.3.5）

- ESM + CJS 双出口，MIT 许可
- 关键词：`standard-schema`、`json-schema`、`convertor`
- 工具链：pnpm、Biome、Vitest、pkgroll

### 与知识库其他条目的关系

JSON Schema 在本库是**输出**；在 [[openapi-ts-request]] 中是**输入**（从 OpenAPI 生成 TS/请求/JSON Schemas）。二者可组成「校验 schema ↔ 契约文档」链路的两端。私有包分发可配合 [[verdaccio]]。

## 相关链接

- 仓库：[standard-community/standard-json](https://github.com/standard-community/standard-json)
- npm：[@standard-community/standard-json](https://www.npmjs.com/package/@standard-community/standard-json)
- Standard Schema 规范：[@standard-schema/spec](https://github.com/standard-schema/standard-schema)
- 同类主题：[[openapi-ts-request]]、[[verdaccio]]
