---
tags: [web-framework, island-architecture, ssr, reactivity, signals, javascript, typescript, frontend, open-source]
created: 2026-06-05
source: "https://github.com/ilhajs/ilha"
---

# Ilha

Ilha（葡萄牙语「岛屿」）是一个**极小、同构（isomorphic）的 Web UI 库**，围绕**岛屿架构（islands architecture）**设计：默认只发送最少的 JavaScript，**只对真正需要交互的部分做 hydration**。它无虚拟 DOM、无需编译器，用 signal 实现细粒度响应式，既能渐进增强服务端渲染的 HTML，也能构建完全自包含的应用。MIT 许可，TypeScript 编写（92.6%），处于 Alpha 阶段，最新版 0.7.0（2026-06-03），GitHub 约 106 stars。

> 官方定位：**ship minimal JavaScript, hydrate only what matters**

## 什么是岛屿架构

「岛屿架构」是一种前端渲染范式：整张页面以**静态 HTML** 为主（服务端渲染、零 JS），只有少数交互组件作为「岛屿」被单独 hydrate（注入 JS 并恢复状态）。相比传统 SPA 把整页都变成一棵需要 hydration 的组件树，岛屿架构只为「岛」付出 JS 成本，从而显著减少首屏脚本体积、加快可交互时间。该理念由 Katie Sylor-Miller / Jason Miller 提出，经 Astro 等框架推广。Ilha 把这一模式做成一个**库**（而非重型框架），核心足够小。

## 核心概念

- **同构 / Universal rendering**：同一份代码在服务端渲染（SSR）、静态生成、混合（hybrid）和边缘（edge）环境运行，开箱即用
- **细粒度响应式（signal）**：基于 signal 的状态，**无虚拟 DOM、无需编译器**；全局 store 由 `alien-signals` 信号引擎驱动
- **SSR + hydration 零闪烁**：服务端渲染后在客户端恢复状态，避免内容跳动（flicker）
- **灵活的作用域**：既可对服务端 HTML 做**渐进增强**，也可构建**完全自包含**的应用
- **文件系统路由**：可选的 Vite 插件，按约定自动生成路由（`@ilha/router`）
- **共享全局状态**：zustand 形态的 store，与核心共用同一套 signal 引擎（`@ilha/store`）
- **后端无关**：可对接任意后端，对 **Nitro** 和 **Hono** 一等支持
- **Prompt 尺寸的源码**：代码库小到**能整体塞进 AI 的上下文窗口**
- **默认类型安全**：全程一等 TypeScript 支持

## 第一个岛屿（Builder API）

Ilha 用**链式 builder** 定义岛屿：`state` 声明状态 → `on` 绑定事件 → `render` 给出模板。事件用 `选择器@事件` 语法（如 `[data-action=increase]@click`），模板用 `html` 标签模板字面量。

在 HTML 中放一个挂载点：

```html
<body>
  <div data-ilha="Counter"></div>
</body>
```

定义并挂载岛屿：

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

`mount()` 会**自动发现页面上所有 `[data-ilha]` 元素**并激活对应的岛屿——这正是「岛屿」自动定位、按需 hydrate 的体现。注意状态以**函数式 getter/setter** 访问：`state.count()` 读、`state.count(v)` 写。

## 三个包

仓库是 monorepo，拆成三个可组合的包：

| 包 | 作用 |
|----|------|
| `ilha` | 核心岛屿 builder——状态、事件、SSR 渲染与 DOM hydration |
| `@ilha/router` | 同构 SPA 路由，支持 SSR，并提供 Vite 文件系统路由插件 |
| `@ilha/store` | zustand 形态的全局 store，由 `alien-signals` 驱动，跨岛屿共享状态 |

## 渲染模式与后端

Ilha 强调「一份代码、多种部署形态」：

- **SSR**：服务端渲染 HTML，客户端 hydrate 岛屿
- **静态生成**：构建期产出纯静态页面
- **Hybrid / Edge**：混合渲染或在边缘运行时执行
- **后端无关**：任意后端可对接，**Nitro / Hono 一等支持**；官方提供 Vite、Hono、Nitro、Elysia 四套模板（`npx giget@latest gh:ilhajs/ilha/templates/<name>`）

## 安装

```bash
npm install ilha
# 或
bun add ilha
```

## 与本知识库的关系

本库属于**前端 Web 框架**领域，与本 wiki 既有的 AI-Agent 主题不同，但有一个有趣的交叉点：

- **「Prompt 尺寸的源码」**——Ilha 把「整库可塞进 AI 上下文窗口」列为正式 feature，与 [[pi]] 的设计哲学呼应：pi 也强调**极小内核**、单体仓库可被模型整体理解。二者都反映出「**为 LLM 友好而刻意保持小体量**」正在成为新库的显性设计目标
- 这种「小到能被一个上下文窗口读完」的趋势，与 [[llm-knowledge-bases]] 关注的**上下文窗口经济学**、[[codegraph]] 用预索引压缩大代码库的探索成本，构成同一问题（让 AI 高效理解代码）的不同解法：要么把库做小，要么把理解过程索引化

## 风险与局限

- **Alpha / 0.x**：官方明确标注「Alpha is live」，API 仍可能变动，不宜用于关键生产
- **生态尚小**：约 106 stars、社区初创（Discord 起步），第三方组件、教程、招聘市场远不及 Astro / Next 等成熟方案
- **范式取舍**：岛屿架构擅长「内容为主 + 局部交互」的站点；对**高度交互的复杂应用**（大量跨岛共享状态、复杂客户端路由）需要更谨慎评估其成熟度
- **builder 风格学习成本**：`state/on/render` 链式 API 与 `选择器@事件` 约定不同于主流 JSX 组件心智，需要适应

## 相关链接

- 官网：[ilha.build](https://ilha.build/) · 仓库：[ilhajs/ilha](https://github.com/ilhajs/ilha)
- 同主题（为 LLM 友好而极简）：[[pi]]
- AI 理解代码 / 上下文经济学：[[codegraph]]、[[llm-knowledge-bases]]
