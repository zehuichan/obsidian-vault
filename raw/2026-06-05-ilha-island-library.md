---
url: "https://github.com/ilhajs/ilha"
clipped: 2026-06-05
title: "ilhajs/ilha — A tiny, framework-free island architecture library"
---

# Ilha

> Alpha is live

**Ilha** is a tiny, isomorphic web UI library built around the islands architecture — ship minimal JavaScript, hydrate only what matters.

**Repository:** [ilhajs/ilha](https://github.com/ilhajs/ilha)
**Website:** ilha.build/
**License:** MIT
**Stars:** 106 | **Forks:** 6 | **Watchers:** 1
**Latest release:** 0.7.0 — Derived DX improvements (Jun 3, 2026) · 14 releases total
**Languages:** TypeScript 92.6% · MDX 6.8% · Other 0.6%
**Topics:** template, library, node, framework, ui, jsx, universal, ssr, vanilla-javascript, full-stack, edge, island, hybrid

## Features

- **Universal rendering** — SSR, static generation, hybrid, and edge rendering out of the box
- **Fine-grained reactivity** — signal-based state with no virtual DOM and no compiler required
- **Flexible scope** — progressively enhance server-rendered HTML, or build fully self-contained apps
- **SSR + hydration** — render on the server, restore state on the client with zero flicker
- **File-system routing** — optional Vite plugin for automatic, convention-based routing
- **Shared global state** — zustand-shaped store backed by the same signal engine as the core
- **Backend agnostic** — integrates with any backend; first-class Nitro and Hono support
- **Prompt-sized source** — small enough to fit the entire codebase into an AI context window
- **Type-safe by default** — first-class TypeScript support throughout

## Quick Navigation

- Website
- Documentation
- Templates
- Discord
- Follow us on X

---

## Getting Started

```bash
npm install ilha
# or with Bun
bun add ilha
```

## Templates

Scaffold a project in seconds with one of the official starters:

| Template | Command | Sandbox |
| --- | --- | --- |
| [Vite](https://github.com/ilhajs/ilha/tree/main/templates/vite) | `npx giget@latest gh:ilhajs/ilha/templates/vite` | [Open](https://stackblitz.com/github/ilhajs/ilha/tree/main/templates/vite) |
| [Hono](https://github.com/ilhajs/ilha/tree/main/templates/hono) | `npx giget@latest gh:ilhajs/ilha/templates/hono` | [Open](https://stackblitz.com/github/ilhajs/ilha/tree/main/templates/hono) |
| [Nitro](https://github.com/ilhajs/ilha/tree/main/templates/nitro) | `npx giget@latest gh:ilhajs/ilha/templates/nitro` | [Open](https://stackblitz.com/github/ilhajs/ilha/tree/main/templates/nitro) |
| [Elysia](https://github.com/ilhajs/ilha/tree/main/templates/elysia) | `npx giget@latest gh:ilhajs/ilha/templates/elysia` | |

---

## Your First Island

Place a mount point anywhere in your HTML:

```html
<body>
  <div data-ilha="Counter"></div>
</body>
```

Define your island and mount it:

```js
import ilha, { html, mount } from "ilha";

const Counter = ilha
  .state("count", 0)
  .on("[data-action=increase]@click", ({ state }) => state.count(state.count() + 1))
  .on("[data-action=decrease]@click", ({ state }) => state.count(state.count() - 1))
  .render(
    ({ state }) => html`
      <p>Count: ${state.count()}</p>
      <button data-action="increase">Increase</button>
      <button data-action="decrease">Decrease</button>
    `,
  );

mount({ Counter });
```

`mount()` auto-discovers every `[data-ilha]` element on the page and activates the matching island.

---

## Packages

This monorepo contains the following packages:

| Package | Description |
| --- | --- |
| [ilha](https://github.com/ilhajs/ilha/blob/main/packages/ilha) | Core island builder — state, events, SSR rendering, and DOM hydration |
| [@ilha/router](https://github.com/ilhajs/ilha/blob/main/packages/router) | Isomorphic SPA router with SSR support and a Vite file-system routing plugin |
| [@ilha/store](https://github.com/ilhajs/ilha/blob/main/packages/store) | Zustand-shaped global store backed by alien-signals — share state across islands |

---

## Community

Have questions or want to share what you're building? Join the Discord to connect with other Ilha developers.

## License

MIT
