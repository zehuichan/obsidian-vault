---
tags: [pascal, 3d, architecture, bim, cad, mcp, agent-skills, react-three-fiber, webgpu, local-first, open-source]
created: 2026-09-18
source: "raw/2026-09-18-pascal-editor.md"
---

# Pascal Editor

Pascal Editor 是一个开源、本地优先的 3D 建筑编辑器：用 React Three Fiber + WebGPU 在浏览器里画墙、楼层、房间和家具，同时通过 CLI 与 MCP 把同一份场景交给 AI Agent 读写。

## 核心概念

- **Local-first 建筑模型**：`npx @pascal-app/cli editor` 即可安装持久本地运行时，项目存在 `~/.pascal/data/pascal.db`，不必先 clone 仓库
- **同一套场景，人和 Agent 都能改**：浏览器 UI 与 MCP 工具操作同一份 node graph；保存后通过本地 scene event stream 同步到编辑器
- **Node + System 架构**：场景是扁平 `Record<id, Node>`；Renderer 只挂占位几何，`WallSystem` / `SlabSystem` 等在 `useFrame` 里按 dirty set 重生几何
- **Viewer / Editor 分离**：viewer 负责渲染与相机；editor 叠加工具、选择层级和直接操纵
- **Plugin = 内置扩展面**：新节点类型、放置工具、inspector 与左侧面板走同一套 `Plugin` manifest，没有第二套内部 API
- **两个公开 Skill**：`pascal-3d` 做安全连接与可验证场景编辑；`furniture-fit` 做有边界的家具占地评估，不臆造层高、开门半径或配送约束

## 详细内容

### 定位

仓库 [pascalorg/editor](https://github.com/pascalorg/editor)，标语是 *open-source 3D architectural editor with a local CLI, MCP tools, and practical workflows for humans and AI agents*。剪藏时约 **24k stars**，MIT 许可。官网文档入口：[editor.pascal.app](https://editor.pascal.app/docs/developers/local-editor)。

它不是通用 3D 建模器，而是面向建筑/室内的 BIM-lite：Site → Building → Level，下面挂 Wall、Slab、Ceiling、Roof、Zone、Scan、Guide，以及门窗灯具等 Item。Topics 含 `architecture`、`bim`、`cad`、`floorplan`、`mcp`、`react-three-fiber`。

这与 [[opencut]]、[[hyperframes]]、[[hypit]] 同属「创作工具 + Agent MCP/skills」路线，只是介质从视频换成了可编辑的 3D 建筑场景。

### 本地运行与 MCP

需要 Node.js **22.13+**：

```bash
npx @pascal-app/cli editor
npm install --global @pascal-app/cli   # 可选，得到 pascal 命令
```

CLI 会：

1. 后台启动编辑器与经过认证的 MCP 服务
2. 自动选不冲突的 loopback 端口
3. 首次启动某版本时下载 web editor runtime，并用包内 digest 校验
4. 把项目写进 `~/.pascal/data/pascal.db`

Agent 侧用稳定连接器 `pascal mcp connect`：它会发现当前动态 MCP 端口；编辑器进程未启动时也会拉起服务。这个连接器本身不需要下载 web runtime，也不需要 Pascal 账号或 API key，不会自动上传项目。

硬约束：**每个本地 CLI 服务同时只接一个活跃 Agent 客户端**。独立本地 HTTP runtime 会在客户端之间共享当前场景；要并行互不干扰，用不同的 `PASCAL_HOME` 和各自的服务进程。托管端点则走会话隔离的 bridge。

常用命令：

| 命令 | 作用 |
|------|------|
| `pascal editor` | 后台启动编辑器 + MCP |
| `pascal mcp connect` | 给 MCP 客户端用的稳定 stdio 连接器 |
| `pascal mcp setup <codex\|claude>` | 写入托管配置，不覆盖已有条目 |
| `pascal mcp config [--json]` | 打印通用 mcp.json |
| `pascal doctor` / `pascal info` | 诊断与路径/运行时信息 |
| `pascal project list` / `pascal open <id>` | 项目管理 |

Agent 建议流程：先读 `pascal://agent-guide`，再 `list_scenes` → `load_scene`，然后改场景。MCP 写入与浏览器看到的是同一份 SQLite。

独立包 `@pascal-app/mcp` 可在 Bun 里头less 跑（无浏览器 / WebGPU / React），把 UI 里的场景变更暴露成工具：`create_wall`、`create_level`、`set_zone`、`place_item`、`cut_opening`、undo/redo、校验与持久化。官方 MCP Registry 条目为 `io.github.pascalorg/editor`（剪藏时 hosted manifest 0.6.1，版本与 npm 包独立追踪）。

### Agent Skills 与各宿主

公开两个 skill（[skills.sh/pascalorg/editor](https://skills.sh/pascalorg/editor)）：

```bash
npx skills add pascalorg/editor \
  --skill pascal-3d \
  --skill furniture-fit
```

| Skill | 用途 |
|-------|------|
| `pascal-3d` | 安全连接本地或托管 MCP，检查/编辑/校验/保存场景，交回可验证 handoff |
| `furniture-fit` | 按给定姿态评估家具占地：碰撞、门洞 keep-out、证据缺口，以及一个有边界、考虑 blocker 的下一步；不声称层高、开门摆动或配送检查 |

Skill 会先读已连接 MCP 的 tool schema，再决定用不用可选字段。仓库源码里有、已安装/托管版本没有的能力，应报告较窄结果，而不是假装源码输入都可用。工具型动作必须连上 Pascal MCP；OpenAI directory 提交因此要走 **With MCP**，并带上生产托管端点。

各通道状态（skills README，2026-09-14）：

| 通道 | 状态 |
|------|------|
| skills.sh | 自动索引，可装 |
| Claude Code plugin | Git marketplace 可装；未提交 Anthropic 目录 |
| Codex / Cursor Agent Plugin | 仓库可装；已上 [Cursor marketplace](https://cursor.com/marketplace/pascal)；0.1.10 修正包仍待市场刷新 |
| Gemini CLI extension | 有 `gemini-extension.json`；gallery 待仓库 topic |
| Official MCP Registry | 已发布 |
| ClawHub / OpenClaw | 未发布，卡在 MIT-0 发行条款 |

Claude / Codex：

```text
/plugin marketplace add pascalorg/editor
/plugin install pascal-agent-skills@pascal
```

插件带两个 server：本地 `pascal mcp connect`，以及托管 `pascal-hosted`（`https://editor.pascal.app/api/mcp`）。Claude 可选填 API key 进系统钥匙串，留空即纯本地。Claude Code 2.1.258 会同时加载 `pascal mcp setup claude` 写的 user-scoped server 和插件自带 server，需先 `claude mcp remove --scope user pascal`，否则违反「一服务一客户端」。

Cursor marketplace 装的是仓库 `skills/`：本地命令钉死 `npx --yes --package=@pascal-app/cli@1.0.0 pascal mcp connect`，不要求全局 `pascal`，也不下载 web editor。托管侧用浏览器 OAuth 登录，不走 API key。只做本地工作时连 `pascal`、断开 `pascal-hosted`。`@pascal-app/cli@1.0.0` 提供 `furniture-fit` 依赖的只读 `check_collisions.candidate`。

这套分发方式是 [[agent-skills]] 的完整落地：同一 `skills/` 目录被 skills.sh、Claude plugin、Cursor plugin、Codex、Gemini extension 复用，而不是为每个宿主重写工作流。

### 包与状态

Turborepo monorepo，发布顺序：`core` → `viewer` → `editor` → `nodes` → `mcp` → `ifc-converter` → `cli`。

| 包 | 职责 |
|----|------|
| `@pascal-app/core` | Zod node schema、Zustand 场景、registry 合约、空间查询、mitt 事件总线；`core/capture` 做版本化 capture manifest |
| `@pascal-app/viewer` | R3F 渲染、相机/控件、后处理；`viewer/capture` 含点云、表面网格、设备运动层 |
| `@pascal-app/editor` | 编辑工具、面板、选择、直接操纵 UI |
| `@pascal-app/nodes` | 内置 registry plugin：节点定义、renderer、几何、systems |
| `@pascal-app/cli` | 版本化独立运行时安装与进程管理 |
| `@pascal-app/mcp` | 把场景工具/资源/prompt/本地存储暴露给 MCP host |
| `apps/editor` | Next.js 宿主 |

三套 Zustand store：

| Store | 包 | 内容 |
|-------|----|------|
| `useScene` | core | nodes / root IDs / dirty set / CRUD；IndexedDB persist + Zundo 50 步撤销 |
| `useViewer` | viewer | 当前选中的 building/level/zone、楼层显示（stacked / exploded / solo）、相机模式 |
| `useEditor` | apps/editor | 当前工具、结构层可见性、面板与偏好 |

嵌入用法：先 `loadPlugin(builtinPlugin)`，再挂 viewer。

### 场景内核

节点都继承 `BaseNode`（`id`、`type`、`parentId`、`visible`、可选 `camera` / `metadata`）。层次逻辑上是树，存储上是扁平字典，用 `parentId` + `children` 还原。

数据流：

```
用户操作 / MCP 工具
  → useScene.createNode / updateNode
  → 节点入 store，标 dirty
  → React 重绘 NodeRenderer，useRegistry 登记 Three.js 对象
  → System 在 useFrame 里按 dirty set 更新几何
  → 清 dirty
```

核心 systems：`WallSystem`（倒角 + CSG 门窗开洞）、`SlabSystem`、`CeilingSystem`、`RoofSystem`、`ItemSystem`。Viewer 还有 `LevelSystem` / `ScanSystem` / `GuideSystem`。空间网格负责地板/墙面放置校验和板面标高。

编辑器工具：Select、Wall、Zone、Item、Slab。选择沿 `Site → Building → Level → Zone → Items` 分层，每层有自己的 hover/click 策略。

### 插件

新节点种类不要改内置包，按 [Create a plugin](https://editor.pascal.app/docs/developers/plugins) 出 plugin：schema、3D/2D 渲染、放置工具、inspector 参数、左侧栏面板，与内置同一 `Plugin` 形状。参考实现：[pascalorg/plugin-trees](https://github.com/pascalorg/plugin-trees)（程序化树、花、草 + presets 面板）。CLI 文档写明当前发行版还不能从 GitHub/npm 装第三方 plugin 进托管 runtime。

### 技术栈与开发

React 19 + Next.js 16、Three.js WebGPU、React Three Fiber + Drei、Zustand、Zod、Zundo、three-bvh-csg、Turborepo、Bun。

```bash
bun install
bun dev          # 必须在仓库根目录，带上 package watchers
# 编辑器：http://localhost:3002
```

## 相关链接

- [[agent-skills]] — Skill 规范与渐进加载；Pascal 用同一 `skills/` 喂 skills.sh / Claude / Cursor / Codex / Gemini
- [[opencut]] — 另一条「开源创作编辑器 + MCP」路线，介质是视频时间线
- [[hyperframes]] — Agent skills + CLI 驱动的 HTML→视频框架，对照 Pascal 的场景 MCP
- [[open-design]] — 把编码 Agent 接到设计工作流；Pascal 则接到可编辑 3D 建筑模型
- [[excalidraw]] — 2D 白板与开放场景格式；Pascal 是 3D 建筑 node graph
- [[codegraph]] — 另一类本地 MCP：代码符号图 vs 建筑场景图
- [[mastra]] — TypeScript Agent 框架侧的 MCP / tools 对照

## 参考

- GitHub: https://github.com/pascalorg/editor
- 本地运行文档: https://editor.pascal.app/docs/developers/local-editor
- 插件文档: https://editor.pascal.app/docs/developers/plugins
- skills.sh: https://skills.sh/pascalorg/editor
- Cursor marketplace: https://cursor.com/marketplace/pascal
- 原始剪藏: `raw/2026-09-18-pascal-editor.md`
