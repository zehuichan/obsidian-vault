---
tags: [nodejs, npm, devops, registry, private-packages, open-source]
created: 2026-07-05
source: "raw/2026-07-05-verdaccio-private-npm-registry.md"
---

# Verdaccio

Verdaccio 是一个轻量级的 Node.js 私有 npm 代理仓库，零配置即可本地运行，自带小型数据库，可代理 npmjs.org 等上游 registry 并缓存已下载模块。

## 核心概念

- **本地私有 registry**：无需完整数据库即可启动，适合团队或单机私有包托管
- **代理 + 缓存**：作为 npmjs.org 等公共 registry 的代理层，下载过的包会本地缓存，降低延迟并具备有限 failover
- **多 registry 聚合**：可将组织内多个 registry 链式接入，客户端只需指向单一 endpoint
- **覆盖公共包**：可在本地发布与第三方同名的修改版包，用于 fork 或内部 patch
- **可扩展存储**：社区插件支持 S3、Google Cloud Storage 等后端，也可自写 storage plugin
- **主流包管理器兼容**：npm、yarn、pnpm 均可直接配置 registry 指向 Verdaccio

## 详细内容

### 适用场景

Verdaccio 解决的是「npm 生态下的私有包与 registry 基础设施」问题：

1. **企业私有包**：在不把代码发布到 npm 公共仓库的前提下，仍用熟悉的 `npm publish` / `npm install` 工作流
2. **CI / 多机安装加速**：多台构建机共享同一缓存层，减少重复拉取
3. **registry 统一入口**：内部 npm 镜像、私有源、公共源通过一个 Verdaccio 实例聚合
4. **依赖 override**：对特定第三方包使用内部修改版，而不改 package name

### 安装与运行

全局安装：

```bash
npm install --global verdaccio
```

启动后默认监听本地端口（常见为 4873），客户端通过 `.npmrc` 或 `npm config set registry` 指向该地址即可。

### 部署方式

官方提供多种运维路径：

| 方式 | 命令 / 说明 |
|------|-------------|
| npm 全局 | `npm install --global verdaccio` |
| Docker | `docker pull verdaccio/verdaccio` |
| Kubernetes | `helm install verdaccio/verdaccio` |

Docker 与 Helm 支持使其适合容器化与 K8s 集群部署，而不仅限于开发机本地使用。

### 与前端 / Node 生态的关系

Verdaccio 被 nx、pnpm、create-react-app、Angular CLI、Storybook 等工具链广泛使用，是 Node.js  monorepo 与前端工程化里常见的私有 registry 方案。它与 [[codemirror]]、[[ilha]] 等前端库无直接耦合，但同为 JavaScript 开发生态的基础设施层组件。

在 Agent / LLM 工作流场景中，若团队通过 [[agent-skills]]、[[openspec]] 等规范维护内部 npm 包或 CLI 工具，Verdaccio 可作为这些私有 artifact 的发布与安装入口，避免依赖公共 npm。

## 相关链接

- 官网：[Verdaccio](https://www.verdaccio.org/)
- 文档：Getting Started、Docker、Configuration（见官网 Docs）
- 同类主题：[[agent-skills]]（内部 skill 包分发）、[[openspec]]（CLI / 规范工具链）
