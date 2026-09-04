---
url: "https://github.com/posva/pinia-colada"
clipped: 2026-09-04
title: "Pinia Colada"
---

# Pinia Colada

> The missing data fetching layer for Vue. Built on top of Pinia

Pinia Colada makes data fetching in Vue applications a breeze. It's built on top of Pinia and takes away all of the complexity and boilerplate that comes with fetching data. It's fully typed and tree-shakeable, and it's built with the same principles as Pinia and Vue: It's approachable, flexible, powerful and can be progressively adopted.

> [!TIP]
> This is a feature-complete version of the exercises from Mastering Pinia. If you would like to learn how it started and become an expert in Vue state management, check it out!

## Features

- ⚡️ Automatic caching: Smart client-side caching with request deduplication
- 🗄️ Async State: Simplified async state management
- 🔌 Plugins: Powerful plugin system
- ✨ Optimistic Updates: UI that updates before the server responds
- 💡 Sensible defaults: Works well out of the box while remaining fully configurable
- 🧩 Out-of-the box plugins: Auto refetch, delay loading, and more
- 📚 Typescript Support: Best-in-class TypeScript support
- 💨 Small Bundle Size: A baseline of ~2kb and fully tree-shakeable
- 📦 Zero Dependencies: No dependencies other than Pinia
- ⚙️ SSR: Out of the box server-side rendering support

> [!NOTE]
> Pinia Colada is always trying to improve and evolve.
> Feedback regarding new and existing options and features is very welcome!
> Contribution to documentation, issues, and pull requests are highly appreciated.

## Installation

```sh
npm install pinia @pinia/colada
```

Install the plugins for the features you need:

```ts
import { createPinia } from 'pinia'
import { PiniaColada } from '@pinia/colada'

app.use(createPinia())
// install after pinia
app.use(PiniaColada, {
  // optional options
})
```

## Usage

The core of Pinia Colada is the `useQuery` and `useMutation` functions. They are used to read data and write it respectively. Here's a simple example:

```vue
<script lang="ts" setup>
import { useRoute } from 'vue-router'
import { useMutation, useQuery, useQueryCache } from '@pinia/colada'
import { patchContact, getContactById } from '~/api/contacts'

const route = useRoute()
const queryCache = useQueryCache()

const { data: contact, isPending } = useQuery({
  // unique key for the query in the cache
  key: () => ['contacts', route.params.id],
  query: () => getContactById(route.params.id),
})

const { mutate: updateContact, isLoading } = useMutation({
  mutation: patchContact,
  async onSettled({ id }) {
    // invalidate the query to refetch the data of the query above
    await queryCache.invalidateQueries({ key: ['contacts', id], exact: true })
  },
})
</script>

<template>
  <section>
    <p v-if="isPending">Loading...</p>
    <ContactCard
      v-else
      :key="contact.id"
      :contact="contact"
      :is-updating="isLoading"
      @update:contact="updateContact"
    />
  </section>
</template>
```

Learn more about the core concepts and how to use them in the documentation.

## License

MIT

---

# Official site extras (pinia-colada.esm.dev)

Homepage: https://pinia-colada.esm.dev/

Additional homepage features not listed in the README:

- 🔄 Data Loaders: Official support for Vue Router Data Loaders

## Installation (docs)

Install Pinia Colada alongside Pinia:

```bash
npm i @pinia/colada
```

Then, install the plugin in your application:

```ts
import { createApp } from 'vue'
import App from './App.vue'
import { createPinia } from 'pinia'
import { PiniaColada } from '@pinia/colada'

const app = createApp(App)
const pinia = createPinia()
app.use(pinia)
app.use(PiniaColada, {
  // Optionally provide global options here for queries
  queryOptions: {
    gcTime: 300_000, // 5 minutes, the default
  },
})
```

### Pinia Colada Devtools

```bash
npm i -D @pinia/colada-devtools
```

Then put the component in your `App.vue` (root component):

```vue
<script setup lang="ts">
import { PiniaColadaDevtools } from '@pinia/colada-devtools'
</script>

<template>
  <main>
    <!-- Your app content here -->
  </main>

  <PiniaColadaDevtools />
</template>
```

By default, the devtools are stripped off in production builds. Use `PiniaColadaProdDevtools` if you want to keep them in production.

### Plugins

```ts
import { PiniaColada, PiniaColadaQueryHooksPlugin } from '@pinia/colada'

app.use(PiniaColada, {
  plugins: [
    PiniaColadaQueryHooksPlugin({
      // ...
    }),
  ],
})
```

Other plugins must be installed separately (e.g. Retries, Loading delay, Auto refetch) and can also be created directly in your project for any custom behavior like offline, cache normalization, etc.

Official plugins:

- Query Hooks (built-in)
- Auto refetch
- Retry
- Delay
- Cache persister

## Queries (docs)

Queries manage asynchronous state declaratively, allowing you to focus on the state, its status, and any potential errors. They automatically deduplicate multiple requests and cache results.

They are designed to read data from asynchronous sources, such as handling `GET` requests in a REST API but they can be used along any function returning a Promise (GraphQL, WebSockets, etc). For writing or mutating data, consider using mutations.

Queries are created with `useQuery()` and can be combined with `defineQueryOptions()` for organization and typing. There is also `defineQuery()` to combine queries with extra state, similar to a tiny Pinia store.

```vue
<script setup lang="ts">
import { useQuery } from '@pinia/colada'

const {
  state,
  asyncStatus,
  refresh,
  refetch,
  error,
  data,
  status,
  isLoading,
  isPending,
  isPlaceholderData,
} = useQuery({
  key: ['todos'],
  query: () => fetch('/api/todos').then((res) => res.json()),
})
</script>
```

All queries require two properties:

- A unique `key` that defines the query in the cache
- A `query` function that retrieves (e.g. fetches) the data

`state` contains:

- `data`: the data returned by the query
- `error`: the error returned by the query. It's `null` if the query was successful
- `status`: `'pending'` → `'success'` or `'error'`

`asyncStatus`: `'idle'` or `'loading'` if the query is currently being fetched.

`state.status` is the status of the data itself, while `asyncStatus` is the status of the query call.

- `refresh()`: manually triggers the query, deduplicates requests, and reuses the cached data if it's still fresh
- `refetch()`: manually triggers the query, ignoring the cache, and fetching the data again

Using external properties (e.g. route params) in queries: add them to the `key` as a function:

```ts
const route = useRoute()
const { state } = useQuery({
  key: () => ['contacts', route.params.id as string],
  query: () =>
    fetch(`/api/contacts/${route.params.id}`).then((res) => res.json()),
})
```

Organizing queries with `defineQueryOptions()` and key factories is recommended as projects grow.

Pausing queries: use `enabled` when required data (e.g. a route param) is not always present. Warning: consuming a query within a store makes the query immortal (stores are never destroyed). Prefer `useQueryCache()` if you need store-level access.

TypeScript: use the `state` object to narrow `data` and `error` types via `status`.

## Query Keys (docs)

Query keys identify a query in the cache. Next to the `query` function, the `key` is the most important property.

Static keys are an array of serializable properties, e.g. `['products']`.

Anything serializable is valid. These are all different keys:

- `['products', 1]`
- `['products', '1']`
- `['products', { id: 1 }]`
- `['products', { id: 1, type: 'book' }]`

Order in objects does not matter. Order in arrays does.

Dynamic keys that depend on reactive variables must be a getter function, `ref`, or `computed`:

```ts
key: () => ['products', route.params.id]
```

When creating query keys, make your key depend on any variable used in your `query` function.

Keys are hierarchical. You can invalidate a parent key to invalidate all children:

```ts
queryCache.invalidateQueries({ key: ['products', productId.value] })
```

Rules:

- `['doc', 2]` and `['doc', '2']` are different keys
- Within objects, `undefined` is stripped out but `null` is not
- Arrays are also partially matched

Key factories + `defineQueryOptions()` tag keys with inferred data types for type-safe cache access.

## Mutations (docs)

Mutations trigger and track the status of an async operation meant to have side effects. Queries read; mutations write. In REST terms: queries usually handle `GET`, mutations handle `POST`/`PUT`/`PATCH`/`DELETE`.

```ts
const {
  mutate: createTodo,
  status,
  asyncStatus,
} = useMutation({
  mutation: (todoText: string) =>
    fetch('/api/todos', {
      method: 'POST',
      body: JSON.stringify({ text: todoText }),
    }),
})
```

Reusable mutations: `defineMutation()`.

Execution:

- `mutate`: main method, does not return anything, catches errors
- `mutateAsync`: returns a promise

Hooks: `onMutate`, `onSuccess`, `onError`, `onSettled`. Global hooks can be set through `mutationOptions` when installing `PiniaColada`.

Strategies to keep queries in sync after mutations:

- Invalidate queries that depend on the changed data
- Optimistic updates to update the UI before the mutation is completed

## Query Invalidation (docs)

```ts
const queryCache = useQueryCache()

// Invalidate a specific query and its children
queryCache.invalidateQueries({ key: ['todos'] })
// Invalidate a specific query only
queryCache.invalidateQueries({ key: ['todos'], exact: true })

// Refetch all active queries
queryCache.invalidateQueries()
```

Custom filtering via `predicate`. By default, invalidates all matching queries but only refetches active ones. Pass `'all'` as second parameter to refetch all.

`useQueryCache` can be used in setup, Pinia stores, and router navigation guards (anywhere `inject()` works). Cannot be invoked in the global scope or within a component method.

Awaiting `invalidateQueries` in mutation hooks delays the mutation's `asyncStatus` until related queries have been refetched.

## Optimistic updates (docs)

Two approaches:

### Via the cache

Update the cache directly in `onMutate`, cancel related queries, rollback in `onError` if the cache value is still the optimistic one, invalidate in `onSettled`.

Pattern:

1. `getQueryData` to snapshot old value
2. `setQueryData` with optimistic new value
3. `cancelQueries` to prevent stale overwrites
4. Return `{ old, new }` for later hooks
5. `onError`: rollback only if cache still equals the optimistic value
6. `onSettled`: `invalidateQueries`
7. `onSuccess`: optionally patch with server response

### Via the UI

Show `variables` of the in-flight mutation next to the query list. When mutation is not collocated, give the mutation a `key` and read it via `useMutationCache().getEntries()`.

## Server Side Rendering (docs)

Pinia Colada relies on Pinia for SSR. Custom data structures require special serialization. Handled automatically in the Nuxt module; custom SSR setups must serialize and revive the cache tree (`serializeQueryCache` / `hydrateQueryCache` / `isQueryCache`).

Garbage collection on the server: default `setTimeout` for `gcTime` is undesirable (keeps Node alive, retains memory). Use `PiniaColadaSSRNoGc` and clear the cache after each render:

```ts
app.use(PiniaColada, {
  plugins: import.meta.env.SSR ? [PiniaColadaSSRNoGc()] : [],
})

// after rendering each request:
useQueryCache(pinia).caches.clear()
```

Lazy queries: `enabled` can skip fetch on the server (`typeof document !== 'undefined'`).

## Nuxt (docs)

Dedicated module: `@pinia/colada-nuxt`.

| Feature | Nuxt Native | Pinia Colada |
| --- | --- | --- |
| Best for | Simple page-level data | Complex app-wide state |
| SSR | `await` required | Automatic via `onServerPrefetch` |
| Parallel requests | Manual via `Promise.all` | Automatic |
| Caching | Manual via `getCachedData` | Automatic + deduplication + GC |
| Mutations | Manual | Built-in `useMutation` |
| Optimistic updates | Manual | Built-in |
| Scope | Pages (prop drilling) | Any component |
| Stale-while-revalidate | Manual | Built-in |

When to use Nuxt native `useFetch`/`useAsyncData`:

- Simple page-specific data that isn't shared
- One-off API calls without complex caching
- Single requests without parallel fetching

When to use Pinia Colada:

- Data shared across multiple components or pages
- Automatic cache invalidation and garbage collection
- Parallel requests within a component
- Cache persistence
- Mutations with optimistic updates
- Complex apps with interdependent data
- Deduplication and stale-while-revalidate

Installation:

```bash
npm install @pinia/colada
npx nuxi module add @pinia/colada-nuxt
npx nuxi module add pinia
```

Configure via `colada.options.ts` at project root.

SSR without await: `useQuery` uses `onServerPrefetch` internally. Use `await refresh()` only when you want to block client-side navigation until data loads.

Migration:

- `pending` → `isPending`
- Add a unique `key`
- Wrap the fetch in `query`
- Remove `await` (SSR works automatically)

Same `key` = same cached data, no props needed across components.

## Links

- Repository: https://github.com/posva/pinia-colada
- Docs: https://pinia-colada.esm.dev/
- npm: `@pinia/colada`
- License: MIT
- Author: Eduardo San Martin Morote (posva)
- Topics: cache, data-fetching, mutation, pinia, pinia-plugin, plugin, query, swr, vue
