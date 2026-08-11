---
tags: [diagram, whiteboard, open-source, frontend, collaboration, json]
created: 2026-08-11
source: "raw/2026-08-11-excalidraw.md"
---

# Excalidraw

开源的手绘风格虚拟白板：无限画布上画流程图、线框与任意草图，支持协作、端到端加密，并以开放的 `.excalidraw` JSON 格式导出。

## 核心概念

- **手绘风格白板**：无限画布 + 矩形/圆/菱形/箭头/自由绘等工具，强调可读的草图感而非精密 CAD
- **开放格式**：导出 `.excalidraw` JSON，可再导入网页端、VS Code 扩展或自建集成
- **编辑器 vs 托管应用**：`@excalidraw/excalidraw` npm 包可嵌入任意 React 应用；[excalidraw.com](https://excalidraw.com) 是同仓 showcase，额外提供 PWA、实时协作、E2E 加密、本地优先自动保存与只读分享链接
- **本地优先提醒**：浏览器存储可能被清理，重要图应定期另存为文件

## 详细内容

### 编辑器能力（npm 包）

免费开源；暗色模式；可定制；图片与 shape library；i18n；导出 PNG / SVG / 剪贴板；箭头绑定与标签；撤销重做；缩放与平移。

### 嵌入集成

```bash
npm install react react-dom @excalidraw/excalidraw
```

文档：https://docs.excalidraw.com/docs/@excalidraw/excalidraw/installation

常见集成面：VS Code 扩展、Obsidian Excalidraw 插件，以及 Google Cloud / Meta / Notion / CodeSandbox 等产品内嵌。

### 与 Agent 出图

自然语言生成 `.excalidraw` 文件可用 [[excalidraw-diagram-generator]] Skill：输出可直接拖到 excalidraw.com 或 VS Code 扩展打开。Skill 约定属于 [[agent-skills]] 中的 Generator 模式。

## 相关链接

- [[excalidraw-diagram-generator]] — 从自然语言生成 `.excalidraw` 的 Agent Skill
- [[agent-skills]] — Skill 规范与 Generator 等设计模式
- [excalidraw.com](https://excalidraw.com)
- [GitHub](https://github.com/excalidraw/excalidraw)
- [文档](https://docs.excalidraw.com)
