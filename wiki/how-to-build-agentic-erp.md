---
tags: [agentic-ai, erp, enterprise-ai, how-to, mcp]
created: 2026-06-06
source: "综合整理自 [[agentic-erp-landing-cases]] + 工程实践"
---

# 搭建 Agentic ERP 实操指南

在已有 ERP 系统上叠加 Agentic 能力的工程方法：补齐五层技术栈、按成熟度四级爬坡、内建治理，并从一个高 ROI 低风险场景切入。

## 核心概念

- **不是「买个 Agent 插上去」**，而是给 ERP 补五层能力 + 分阶段演进
- **五层技术栈**：工具层 → 知识/记忆层 → 推理层 → 编排层 + 横切的**治理层**
- **按成熟度爬坡**：L1 只读助手 → L2 半自动 workflow → L3 自治 Agent → L4 多 Agent（绝大多数失败源于直接做 L3/L4）
- **数据质量是硬上限**：主数据脏，Agent 推理系统性出错且难发现
- **治理从第一天内建**：Agent 能下单/转账后，权限、审批、审计、回滚是底线

## 五层技术栈

你的 ERP 提供最底层的「数据 + 业务 API」，Agentic 化是在它上面叠：

| 层 | 作用 | 关键动作 |
|----|------|----------|
| 工具层 | 让 LLM 能「操作」ERP | 把 API 封装成 MCP Server / Function Calling 工具，定义入参出参与权限 |
| 知识/记忆层 | 让 Agent 理解你的业务对象 | RAG 接业务文档；进阶做业务本体/知识图谱（参考 [[codegraph]] 给代码建图谱） |
| 推理层 | 多步决策 | 选模型：数据合规优先 DeepSeek/Qwen，复杂推理用 Claude/GPT |
| 编排层 | 串成业务闭环 | 先单 Agent ReAct，成熟后多 Agent 协同 |
| 治理层（横切） | 风险控制 | 权限边界、审批、审计日志、回滚 |

工具层的设计思路与 [[agent-skills]] 一致：把能力做成可被 Agent 发现、按需调用的标准单元。

## 落地路线：成熟度四级爬坡

| 阶段 | 做什么 | 风险 | 建议 |
|------|--------|------|------|
| L1 只读助手 | 自然语言查数 / 生成报表 | 极低 | 第一步，2–4 周见效、建立信任 |
| L2 半自动 workflow | 自动对账、生成审批/催收草稿、异常告警 | 中 | Agent 出草稿，人确认（human-in-the-loop） |
| L3 自治 Agent | 选一个高 ROI 闭环让它自己跑 | 高 | 限定授权边界 + 强审计后再上 |
| L4 多 Agent 自治 | 跨域协同 | 很高 | 现阶段别碰 |

## 四个前提（落地壁垒的反向 checklist）

1. **数据质量先行**：主数据去重、编码统一，否则 Agent 基于错误前提决策
2. **隐性规则显性化**：把「老员工才知道、没人敢删」的定制规则梳理成文档
3. **Governance 内建**：仅 21% 企业有成熟治理；权限/审批/审计/回滚不能事后补
4. **人机协作建信任**：先草稿后执行，靠成功案例逐步放权

## 三条切入路径

- **商业 ERP（SAP/Oracle/用友/金蝶）** → 优先用厂商原生 Agent（Joule / Fusion Agent / 用友数智员工 / 金蝶苍穹），别自己造轮子
- **自研 / 深度定制 ERP** → 自己搭：API 封装成 MCP Server，接 LLM + 编排框架，自建知识层与治理层（见下）
- **旧版动不了 / 跨多系统** → Agent Overlay：不动现有 ERP 在外叠一层；无 API 环节用 RPA（UiPath/AA）补位

## 自研 ERP 详细方案

自研 ERP 的最大优势：对数据库和 API 有完全控制权，可自由定义工具层。

### 技术选型

| 层 | 推荐 |
|----|------|
| 模型 | DeepSeek-V3 / Qwen（成本、数据合规）；复杂推理 Claude / GPT。统一走 OpenAI 兼容 API |
| 工具层 | MCP Server（Python/TS SDK），按业务域分组 |
| 知识层 | pgvector / Chroma / Milvus 做 RAG；语义层文档描述 schema |
| 编排 | LangGraph（状态机式、可控）或 Dify（低代码、快） |
| 前端 | 嵌入 ERP 的对话框 / 企业微信 / 钉钉机器人 |

### 工具层设计（最关键）

把 ERP API 封装成 MCP 工具，给 LLM 看的 `description` 要写清楚业务语义。**只读工具与写工具分级**，写工具强制审批：

```json
{
  "name": "query_sales_report",
  "description": "查询指定区域和时间段的销售额、毛利率，支持同比/环比",
  "parameters": {
    "region": "区域编码或名称（华东/华南/...）",
    "period": "YYYY-MM",
    "compare": "yoy | mom | none"
  }
}
```

```json
{
  "name": "create_purchase_order",
  "description": "创建采购订单草稿；提交后需采购经理审批，不直接生效",
  "requires_approval": true,
  "parameters": { "supplier_id": "...", "items": "[...]" }
}
```

### Text-to-SQL 的取舍

自然语言查数有两条路：

| 方案 | 优点 | 风险 | 适用 |
|------|------|------|------|
| 参数化查询工具 | 安全、可控、性能稳 | 覆盖面有限 | MVP 首选 |
| Text-to-SQL（LLM 生成 SQL） | 灵活、覆盖广 | 错误 SQL、权限绕过、性能 | 成熟后引入，须配只读副本 + SQL 校验 + 表白名单 + 强制 LIMIT |

建议：MVP 用参数化工具，覆盖 80% 高频问题；后续对长尾需求引入受限 Text-to-SQL。

### 治理层

- **权限**：Agent 调 API 时带上**操作者身份**，复用 ERP 现有 RBAC，绝不给 Agent 超级权限
- **审批**：写操作走「Agent 生成 → 人确认 → 执行」
- **审计**：每次工具调用记日志（who / what / when / result / Agent 推理过程）
- **回滚**：写操作要可逆或设补偿事务

## 第一个场景：自然语言查询经营报表（L1）

最适合自研 ERP 起步——只读、零风险、高频、ROI 直观。

- **交互**：用户问「上月华东区销售额和毛利率，对比去年同期」→ Agent 理解意图 → 调 `query_sales_report(region, period, compare)` → ERP 返回数据 → LLM 组织成自然语言 + 表格
- **要做的事**：
  1. 封装 3–5 个只读报表工具（销售 / 库存 / 应收 / 财务）
  2. 写语义层文档：字段含义、区域编码映射、指标口径
  3. 单 Agent ReAct 循环
  4. 接入 ERP 对话框或企业微信机器人
- **交付周期**：2–4 周
- **下一步**：跑通后演进到 L2（自动对账、催收/审批草稿）

## 相关链接

- [[agentic-erp-landing-cases]] — 全球 Agentic ERP 落地案例与厂商战略
- [[agent-skills]] — Agent Skill 规范与构建方法
- [[codegraph]] — 给代码建知识图谱的 MCP 服务（业务知识图谱的参照）
- [[hermes-agent]] — 开源 Agent 架构与工具调用
