---
tags: [openapi, typescript, codegen, axios, react-query, apifox, frontend, open-source]
created: 2026-07-28
source: "raw/2026-07-28-openapi-ts-request.md"
---

# openapi-ts-request

openapi-ts-request 是根据 Swagger2 / OpenAPI3 / Apifox 文档自动生成 TypeScript/JavaScript 类型、请求客户端函数、mock、枚举翻译、react-query/vue-query 与 JSON Schemas 的代码生成工具（openapi-ui, MIT）。

## 核心概念

- **文档驱动生成**：从 OpenAPI/Swagger/Apifox 定义一键产出类型与请求函数，减少手写 API 层
- **客户端无关**：默认 axios，可自定义指向 Fetch、UniApp、Taro、Node.js、XHR 等任意 request 封装
- **多入口使用**：CosmiConfig 配置文件、Node/TS 脚本调用 `generateService`、npx/dlx、全局 CLI `openapi`
- **过滤与增量控制**：按 tags/paths include/exclude，支持中文 tag 英译、按模块拆分类型文件
- **周边产物**：可选生成 react-query/vue-query、字段 label、枚举翻译、JSON Schemas、mockjs mock

## 详细内容

### 解决什么问题

前端项目对接后端时，手写请求函数与类型容易与文档漂移。openapi-ts-request 把「读文档 → 写类型 → 写 request」固化为生成步骤：输入 schema（URL/本地 JSON/YAML，或 Apifox `projectId` + token），输出 `serversPath` 下的 `index.ts`、`types.ts` 及按 tag 拆分的接口文件。

### 快速接入

安装为开发依赖后，根目录建 `openapi-ts-request.config.ts`（亦支持 cosmiconfig 常见变体）：

```ts
import { defineConfig } from 'openapi-ts-request';

export default defineConfig({
  schemaPath: 'http://petstore.swagger.io/v2/swagger.json',
  serversPath: './src/apis',
});
```

`package.json` 增加 `"openapi": "openapi-ts"`，执行 `npm run openapi`。也可多 schema 数组配置，分别输出到不同目录。无配置时可直接：

```bash
npx --package=openapi-ts-request -- openapi -i ./openapi.json -o ./apis
```

### 关键配置面

| 能力 | 配置要点 |
|------|----------|
| 请求库 | `requestLibPath` / `requestImportStatement` |
| 只出类型 | `isOnlyGenTypeScriptType` |
| Query 层 | `isGenReactQuery` + `reactQueryMode`（react/vue） |
| 过滤 | `priorityRule` + include/exclude Tags/Paths |
| Apifox | `apifoxConfig`（与 `schemaPath` 互斥） |
| 定制命名/模板 | `hook.customFunctionName` 等自定义 Hook |

Hook 可改函数名、类型名、文件名、模板与 OpenAPI 数据预处理；旧版 `operationId` 命名可通过 hook 兼容。

### Mock 与小程序

Mock 基于 mockjs，启动依赖 `@umijs/server`。UniApp 场景推荐自定义 request 函数，或配合 `@uni-helper/axios-adapter`。

### 与知识库其他条目的关系

同属前端/TypeScript 工程化链路：[[ilha]]、[[codemirror]] 偏运行时 UI；本工具偏 API 契约到代码的生成层。若团队用 [[openspec]] 或 [[trellis]] 管理规格与任务，OpenAPI 文档可作为契约源，由本工具落到可调用的 TS 客户端。私有包分发可配合 [[verdaccio]]。

## 相关链接

- 仓库：[openapi-ui/openapi-ts-request](https://github.com/openapi-ui/openapi-ts-request)
- npm：[openapi-ts-request](https://www.npmjs.com/package/openapi-ts-request)
- 使用手册：[Issue #100](https://github.com/openapi-ui/openapi-ts-request/issues/100)
- 前身致谢：[openapi2typescript](https://github.com/chenshuai2144/openapi2typescript)
- 同类主题：[[verdaccio]]、[[openspec]]、[[ilha]]
