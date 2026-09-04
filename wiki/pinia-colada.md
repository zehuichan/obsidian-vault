---
tags: [vue, pinia, data-fetching, query, mutation, cache, ssr, nuxt, typescript, frontend, open-source]
created: 2026-09-04
source: "https://github.com/posva/pinia-colada"
---

# Pinia Colada

Pinia Colada（`@pinia/colada`）是 Vue 的**数据获取层**：建在 [Pinia](https://pinia.vuejs.org) 之上，用 `useQuery` / `useMutation` 处理异步读写、客户端缓存、请求去重和乐观更新。作者是 Pinia 作者 Eduardo San Martin Morote（posva），定位为 Mastering Pinia 课程练习的完整成品版。MIT，基线约 2kb、可 tree-shake，除 Pinia 外零运行时依赖。剪藏时仓库约 2.2k stars，npm 最新稳定版 1.4.2（2026-07-15），peer 依赖 Pinia 4.x。

> 官方定位：**The missing data fetching layer for Vue. Built on top of Pinia**

## 核心概念

- **Query / Mutation 分离**：`useQuery` 读（任意返回 Promise 的函数，不限于 REST GET）；`useMutation` 写（POST/PUT/PATCH/DELETE 等有副作用的操作）
- **按 key 缓存**：每个 query 必须有唯一 `key`（可序列化数组）；相同 key 共享同一条缓存，自动去重
- **双状态**：`state.status`（`'pending' | 'success' | 'error'`，数据本身）与 `asyncStatus`（`'idle' | 'loading'`，这次请求是否在飞）分开，便于区分「从未成功过」和「后台刷新」
- **层级 key + 失效**：key 是树；`invalidateQueries({ key: ['products', id] })` 会失效该前缀下所有子查询
- **乐观更新**：在 `onMutate` 里直接改 cache，失败按快照回滚，完成后 invalidate
- **插件**：安装时注入全局行为（hooks、auto refetch、retry、delay、cache persister）
- **SSR / Nuxt**：缓存可序列化；Nuxt 模块自动 hydration，服务端关掉 GC，查询走 `onServerPrefetch`，不必 `await`

## 安装

必须先装 Pinia，再装 Colada 插件：

```bash
npm install pinia @pinia/colada
```

```ts
import { createPinia } from 'pinia'
import { PiniaColada } from '@pinia/colada'

app.use(createPinia())
app.use(PiniaColada, {
  queryOptions: {
    gcTime: 300_000, // 默认 5 分钟
  },
})
```

开发工具：`@pinia/colada-devtools`，在根组件放 `<PiniaColadaDevtools />`（生产默认剥离；需要线上调试用 `PiniaColadaProdDevtools`）。

Nuxt：

```bash
npx nuxi module add pinia
npx nuxi module add @pinia/colada-nuxt
```

根目录 `colada.options.ts` 传入 `PiniaColadaOptions`。

## Query：声明式读数据

```ts
const { state, asyncStatus, refresh, refetch, data, isPending } = useQuery({
  key: ['todos'],
  query: () => fetch('/api/todos').then((res) => res.json()),
})
```

| 方法 | 行为 |
|------|------|
| `refresh()` | 去重；缓存仍 fresh 则复用 |
| `refetch()` | 忽略缓存强制拉 |

依赖路由 / props 等外部值时，**key 必须是 getter**，且包含 `query` 里用到的所有变量：

```ts
key: () => ['contacts', route.params.id]
```

`enabled` 可暂停查询（例如全局 composable 里路由参数不在时不要发请求）。文档警告：**不要把 query 放进 Pinia store**——store 永不销毁，query 会变成「永生」并持续 watch key。需要在 store / 路由守卫里碰数据，用 `useQueryCache()`。

TypeScript 收窄用返回的 `state` 对象（`status` + `data` + `error` 捆在一起），不要拆开三个 ref 再 `v-if`——Vue 的 Ref 无法被 TS 联动收窄。

## 组织查询：key factory + defineQueryOptions

项目一大，硬编码 key 容易打错，optimistic update / invalidate 也会丢类型。推荐按领域拆文件：

```ts
export const DOCUMENT_QUERY_KEYS = {
  root: ['documents'] as const,
  byId: (id: string) => [...DOCUMENT_QUERY_KEYS.root, id] as const,
}

export const documentByIdQuery = defineQueryOptions(
  ({ id }: { id: string }) => ({
    key: DOCUMENT_QUERY_KEYS.byId(id),
    query: () => getDocumentById(id),
  }),
)

// 组件里
useQuery(() => documentByIdQuery({ id: route.params.docId as string }))

// cache 读写带推断类型
queryCache.getQueryData(documentByIdQuery('some-id').key)
```

`defineQuery()` 则把 query 和额外本地状态捆成「微型 store」，供多组件复用。单处覆盖选项（如 `enabled`）时，把 options 展开进 `useQuery(() => ({ ...base, enabled }))`。

Key 规则：对象字段顺序无关；数组顺序有关；`['doc', 2]` ≠ `['doc', '2']`；对象里 `undefined` 会被剥掉（`{ withComments: undefined }` ≡ `{}`），`null` 保留。

## Mutation：写数据并同步 cache

```ts
const { mutate, mutateAsync, asyncStatus } = useMutation({
  mutation: (text: string) => createTodo(text),
  onSettled: () => queryCache.invalidateQueries({ key: ['todos'] }),
})
```

- `mutate`：不返回值，内部吞错，适合模板绑定
- `mutateAsync`：返回 Promise，自己处理错误或等待完成
- 钩子：`onMutate` / `onSuccess` / `onError` / `onSettled`；全局钩子走安装时的 `mutationOptions`
- 可复用：`defineMutation()`（可带共享 `ref`，或只传 options 对象）

`invalidateQueries` 默认把匹配项标 stale，**只 refetch 仍被组件使用的 active query**；未挂载的等下次激活再拉。第二参传 `'all'` 则全部立刻 refetch。在 hook 里 `await invalidateQueries(...)` 会把 mutation 的 `asyncStatus` 拖到相关 query 刷新完，方便转圈到数据真正就位。

## 乐观更新

**改 cache（推荐，改一处所有订阅自动更新）**：

1. `onMutate`：`getQueryData` 快照 → `setQueryData` 写入乐观值 → `cancelQueries` 防止过期响应盖回来 → `return { old, new }`
2. `onError`：仅当 cache 仍等于乐观值时回滚（避免覆盖别人后来的写入）
3. `onSettled`：`invalidateQueries` 与服务器对齐
4. `onSuccess`：可选，用服务端实体替换临时 id / 占位字段

**改 UI（mutation 和 query 同处时更简单）**：用 mutation 的 `variables` 在列表末尾画一条半透明项；mutation 不在同一组件时给它 `key`，另一边用 `useMutationCache().getEntries({ key })` 读飞行中的状态。

## 插件与生态

安装时 `plugins: [...]`，同 hook 按安装顺序执行。

官方插件：

| 插件 | 说明 |
|------|------|
| Query Hooks | 内置，全局 query 生命周期 |
| Auto refetch | 窗口聚焦 / 重连等自动刷新 |
| Retry | 失败重试 |
| Delay | 推迟 loading，避免闪一下 |
| Cache persister | 缓存持久化 |

另有 `@pinia/colada-devtools`、`@pinia/colada-nuxt`，以及 Vue Router Data Loaders 官方支持。自定义插件可做离线、cache 规范化等。

## SSR 与 Nuxt

自定义 SSR：用 `devalue` 注册 `PiniaColada_TreeMapNode`，`serializeQueryCache` / `hydrateQueryCache`。必须先 `app.use(pinia)` + `app.use(PiniaColada)` 再 hydrate。

服务端不要跑 `gcTime` 定时器（会拖住 Node SSG、跨请求泄漏）。`PiniaColadaSSRNoGc` 强制 `gcTime: false`；每请求结束后 `useQueryCache(pinia).caches.clear()`。Nuxt 模块两件事都替你做了。

与 Nuxt 原生 `useFetch` / `useAsyncData` 的取舍：页面一次性数据用原生；跨组件共享、并行请求、失效 / GC、mutation、乐观更新、SWR 用 Colada。`useQuery` 通过 `onServerPrefetch` 自动等数据，**不必 `await`**；只有想挡住客户端导航时才 `await refresh()`。

## 在生态里的位置

Vue 侧异步数据常见三条路：

1. **手写 Pinia + loading/error**：灵活但样板多，没有统一 cache / 去重
2. **TanStack Query（`@tanstack/vue-query`）**：跨框架 query-core，生态最大；cache 是独立子系统
3. **Pinia Colada**：cache 长在 Pinia 里，API 更贴 Vue / Pinia 心智，体积更小

它不是「又一个 HTTP 客户端」——`query` / `mutation` 只是 Promise 工厂，具体请求仍用 `$fetch`、axios 或 [[openapi-ts-request]] 生成的函数。openapi-ts-request 目前可生成 vue-query 包装；选型 Colada 时生成「纯请求函数」再自己套 `useQuery` 即可。

## 相关链接

- [[openapi-ts-request]] — 从 OpenAPI 生成请求函数与 vue-query；可与 Colada 叠用（生成客户端，Colada 管 cache）
- [[ilha]] — 同属 Vue 生态前端库（岛屿架构，关注少发 JS）
- [[standard-json]] — 请求/响应校验可接 Standard Schema，与 query 函数组合

## 参考

- 仓库：[github.com/posva/pinia-colada](https://github.com/posva/pinia-colada)
- 文档：[pinia-colada.esm.dev](https://pinia-colada.esm.dev/)
- npm：`@pinia/colada`（1.4.2，2026-07-15）
- 课程来源：[Mastering Pinia](https://masteringpinia.com)
- 原始剪藏：`raw/2026-09-04-pinia-colada.md`
