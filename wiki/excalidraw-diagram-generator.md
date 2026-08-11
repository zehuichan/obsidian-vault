---
tags: [agent-skills, excalidraw, diagram, generator, workflow]
created: 2026-08-11
source: "raw/2026-08-11-excalidraw-diagram-generator.md"
---

# Excalidraw Diagram Generator

来自 `github/awesome-copilot` 的 Agent Skill：把自然语言描述编译为可在 [[excalidraw]] 打开的 `.excalidraw` JSON，覆盖流程图、架构图、时序图等九类图。

## 核心概念

- **Generator Skill**：典型 [[agent-skills]] Generator 模式——解析意图 → 选图类型 → 抽结构 → 写 JSON → 给出打开方式
- **九类图**：Flowchart、Relationship、Mind Map、Architecture、DFD、Swimlane、Class、Sequence、ER
- **开放输出**：标准 Excalidraw 文件（`type: excalidraw`, `version: 2`），可在 [excalidraw.com](https://excalidraw.com) 或 VS Code 扩展打开
- **资产齐全**：`templates/` 八种模板、`references/` schema 与元素说明、`scripts/`（箭头、图标库、library 拆分）可选增强架构图

## 详细内容

### 安装

```bash
npx skills add github/awesome-copilot@excalidraw-diagram-generator -g -y
```

skills.sh 安装量约 27.6K（剪藏时）。本机路径：`~/.agents/skills/excalidraw-diagram-generator/`（并复制到 Cursor 等 Agent 技能目录）。

安全说明：skills.sh 上 Gen Agent Trust Hub 标记偏高风险；使用前宜审阅 `SKILL.md` 与 `scripts/`。

### 工作流摘要

1. 判断图类型与复杂度  
2. 按类型抽取步骤 / 实体 / 泳道角色 / 消息等  
3. 生成元素：`rectangle` / `ellipse` / `diamond` / `arrow` / `text`  
4. 文本统一 `fontFamily: 5`（Excalifont）  
5. 保存为 `<name>.excalidraw`，提示用户拖到 [[excalidraw]] 或 VS Code 打开  

布局约定：水平间距约 200–300px；主色 `#a5d8ff`、次色 `#b2f2bb`、中心 `#ffd43b`、警告 `#ffc9c9`；元素过多时建议拆成多张图。

### 触发示例

「画一张 CI/CD 流程图」「生成系统架构 Excalidraw」「做 ER 图 / 时序图 / 思维导图」等。

## 相关链接

- [[excalidraw]] — 白板产品与 `.excalidraw` 开放格式
- [[agent-skills]] — Skill 结构与 Generator 模式
- [skills.sh 页面](https://skills.sh/github/awesome-copilot/excalidraw-diagram-generator)
- [awesome-copilot](https://github.com/github/awesome-copilot)
