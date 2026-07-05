---
tags: [ai-agent, agent-skills, coding-agent, yagni, anti-pattern, minimalism, open-source]
created: 2026-07-05
source: "raw/2026-07-05-ponytail-lazy-senior-dev-ai-agent.md"
---

# Ponytail

由 DietrichGebert 开源的 **AI 编码 Agent 规则集 / 插件**，口号是「the lazy senior dev for your AI agent」：让 Agent 像被凌晨 3 点 on-call 过太多次的 senior dev 一样——少说话，写一行，能跑。它通过一套「决策阶梯（ladder）」约束 Agent 优先 YAGNI、复用现有代码、标准库、原生平台能力和已装依赖，最后才写「刚好够用」的最小 diff；并宣称在 FastAPI + React 仓库的 12 个 feature 任务上，中位数减少 54% 代码、22% token、20% 成本、27% 耗时，同时 100% 保留 validation / error handling / security / accessibility。MIT 许可，支持 Cursor、Claude Code、Codex、Copilot CLI、Gemini CLI、[[pi]] 等 14+ 编码 Agent。

> He says nothing. He writes one line. It works.

## 核心概念

- **问题定义**：Agent 常默认写 50 行自定义抽象，而任务可能只需标准库一行（如 Python `@lru_cache` 替代自研 `CacheManager`）
- **决策阶梯（The Ladder）**：按顺序停在第一个够用的 rung——(1) 是否需要存在 (YAGNI) → (2) 代码库内是否已有 → (3) 标准库 → (4) 原生平台 → (5) 已装依赖 → (6) 能否一行 → (7) 才写最小实现
- **安全不妥协**：压缩代码不等于砍掉 validation、错误处理、安全与无障碍；benchmark 明确这些维度不被简化
- **强度可调**：`lite`（默认，给出更懒替代方案让你选）、`full`（强制阶梯）、`ultra`（YAGNI 极端，一行交付并质疑剩余需求）
- **可审计**：提供 review / audit / debt / gain 命令，把 over-engineering 和 deferred shortcuts 显式化

## 决策阶梯详解

Ponytail 的核心不是「写更少注释」或「删测试」，而是一套**优先级明确的选型顺序**：

1. **YAGNI** —  speculative 需求直接跳过
2. **Reuse** — 项目里已有的 helper、util、pattern 优先
3. **Stdlib** — 语言/运行时内置能力优先
4. **Native** — 如 `<input type="date">` 优于 date picker 库
5. **Installed deps** — 不新增依赖，先用已有的
6. **One-liner** — 能一行就不写十行
7. **Minimum viable** — 以上都不行，才写最短可工作 diff

这与 [[impeccable]] 形成有趣对照：Impeccable 往 Agent 里**加**设计词汇与反模式检测；Ponytail 往 Agent 里**减**不必要的抽象与代码量。两者都通过 [[agent-skills]] 式的插件/规则包注入行为，但优化目标相反——一个管「好不好看」，一个管「要不要写」。

## 命令参考

| 命令 | 作用 |
|------|------|
| `/ponytail lite\|full\|ultra\|off` | 设置强度或关闭 |
| `/ponytail-review` | 审查当前 diff 中的 over-engineering |
| `/ponytail-audit` | 扫描全仓库代码膨胀 |
| `/ponytail-debt` | 收集 deferred shortcuts 到 ledger |
| `/ponytail-gain` | 显示 benchmark 记分板 |
| `/ponytail-help` | 命令速查 |

## 安装（Cursor 等）

Ponytail 以插件/marketplace 形式分发，不同 Agent 安装方式略有差异。以 Claude Code 为例：

```
/plugin marketplace add DietrichGebert/ponytail
/plugin install ponytail@ponytail
```

Pi harness：

```
pi install git:github.com/DietrichGebert/ponytail
```

完整 Agent 列表见 [GitHub README](https://github.com/DietrichGebert/ponytail)。

## Benchmark 数据

在 FastAPI + React 仓库、12 个 feature 任务上的中位数（来源：[ponytail.dev](https://ponytail.dev/)）：

| 指标 | 变化 |
|------|------|
| 代码量 | −54% |
| Token | −22% |
| 成本 | −20% |
| 速度 | +27% |
| 安全相关维度 | 100% 保留 |

## 与同类工具的关系

- **[[headroom]]** — 压缩的是**上下文/token**（工具输出、日志、RAG）；Ponytail 压缩的是**生成的代码本身**
- **[[codegraph]]** — 帮 Agent 少 grep、少读文件；Ponytail 帮 Agent 少写文件里的代码
- **[[openspec]] / [[trellis]]** — 在规划层加结构与规格；Ponytail 在实现层强制最小 diff，可与 spec-driven 流程叠加
- **[[pi]]** — Pi 的「Primitives, not features」是 harness 设计哲学；Ponytail 是注入到 harness 内的编码行为规则

## 相关链接

- [[agent-skills]] — Skill/插件包的规范结构与渐进式加载
- [[impeccable]] — 另一方向的 Agent 行为注入：设计质量而非代码量
- [[pi]] — 支持 ponytail 安装的极简编码 Agent harness
- [[headroom]] — 上下文压缩，与代码压缩互补
- [[trellis]] — 团队级 Agent 脚手架，可与 ponytail 的 minimal-diff 原则组合
