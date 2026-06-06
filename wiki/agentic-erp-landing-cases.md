---
tags: [agentic-ai, erp, enterprise-ai, case-study]
created: 2026-06-06
source: "raw/2026-06-06-agentic-erp-landing-cases.md"
---

# Agentic ERP 落地案例

Agentic ERP 是具备感知、推理、记忆、工具调用与自主执行能力的 ERP 系统，让 AI Agent 从被动问答助手升级为可独立完成业务目标的数字员工；2025–2026 年正值其规模化落地周期。

## 核心概念

- **AI Assistant vs Agentic AI**：前者靠 RAG + LLM 被动查数；后者接收业务目标后自主拆解、联动多系统执行、复盘汇报
- **成熟度四级模型**：Level 1 AI Assistant（~60%）→ Level 2 Workflow AI（~25%）→ Level 3 Autonomous Agent（~12%）→ Level 4 Multi-Agent System（~3%）
- **四大技术成熟条件**：多步推理大模型、Function Calling/MCP 工具调用标准化、ERP 全维度 API 开放、多 Agent 编排框架
- **三大厂商阵营**：传统 ERP 巨头（SAP/Oracle/Microsoft/Workday）、云原生平台（Salesforce/ServiceNow/UiPath/IBM）、AI Native + 国产厂商（Doss/Rillet/用友/金蝶/浪潮）
- **本质变革**：ERP 从「记录人类操作的系统」变为「调度人类与 AI 员工协同工作的操作系统」

## 产业落地逻辑

Gartner 预测 2026 年底 40% 企业应用将内嵌任务型 AI Agent（2025 年不足 5%）。真正 Agentic ERP 需五大能力齐备，但市面多数「AI ERP」仍停留在 Copilot 层。

| 成熟度 | 占比 | 技术特征 | 代表场景 |
|--------|------|----------|----------|
| L1 AI Assistant | ~60% | RAG + LLM，被动响应 | 自然语言查数、智能报表 |
| L2 Workflow AI | ~25% | 规则引擎 + AI，半自动 | 自动对账、智能审批 |
| L3 Autonomous Agent | ~12% | Tool Calling + 自主执行 | 采购 Agent、财务关账 |
| L4 Multi-Agent | ~3% | Agent 间通信 + 分布式决策 | 供应链端到端自治 |

## 三大阵营厂商战略与案例

### 第一阵营：传统 ERP 巨头

**SAP** — 构建 Enterprise World Model + Knowledge Graph 数据护城河，愿景 Autonomous Enterprise。核心产品：SAP Business AI Platform、SAP Autonomous Suite（50+ Joule 智能体、200+ 专项 Agent）。

| 案例 | 效果 |
|------|------|
| Joule Procurement Agent | 采购周期 8→3 天，价格降 4.2%，事务性工作占比 75%→28% |
| Cash Management Agent | 手动现金定位时间减 80%，资金闲置成本降 15% |
| Supply Chain Agent | 异常响应 48h→15min，停工时间减 67% |

**Oracle** — Outcome-based ERP：用户定义结果，Agent 自选流程。Fusion Agentic Applications（Release 26B）将 ERP/HCM/SCM 全面 Agent 化，AI Agent 功能纳入 SaaS 订阅不额外收费。

| 案例 | 效果 |
|------|------|
| Financial Closing Agent | 月结 7→2 天，手工操作减 70%，核对错误率降 85% |
| Supplier Negotiation Agent | 谈判周期 5 天→8 小时，价格降 3.2% |
| Project Management Agent | 延误率 42%→18%，成本超支 38%→12% |

**Microsoft** — Agent 是跨系统连接器，打通 Dynamics 365 + M365 + Teams + Copilot。战略：「不做最好的 Agent，做最好的 Agent 平台」。

| 案例 | 效果 |
|------|------|
| Finance Copilot（应收催收） | 催收时间减 40%，周转天数 72→54 天 |
| Sales Development Agent | 线索转化率 +15.1%，SDR 效率 2.5× |

**Workday** — 将 AI Agent 定位为新型劳动力，愿景「管理人、资金和 AI Agent」。FY2026 交付 17 亿次 AI Actions，1200+ 客户完成 Agent 注册观测（Agent System of Record 领先）。

### 第二阵营：云原生平台

- **Salesforce Agentforce**：客服自证 83% 请求无需人工；Revenue Agent 交叉销售 +19%；Commerce Agent 购物车转化 +14%
- **ServiceNow Now Assist**：「Now on Now」120 天节省 1000 万美元；IT 工单解决 48h→12h，自动解决率 62%
- **UiPath Autopilot**：发票数据提取准确率 >95%，模板变更自适应（传统 RPA 需 2–3 周重写）
- **Automation Anywhere KYC**：银行审核 3–5 天→4 小时，合规覆盖率 78%→100%
- **IBM watsonx**：强监管行业 AI Governance（可解释性、审计追踪）；保险理赔全自动率 70%，处理时间 14→3 天

### 第三阵营：AI Native + 国产

**AI Native**：Doss（自然语言即界面）、Rillet（按交易量收费财务自动化）、DualEntry（零手工录入）、Rimini Street（旧版 ERP Agent Overlay，成本为升级 1/10）

**国产厂商**：

| 厂商 | 战略 | 标杆数据 |
|------|------|----------|
| 用友 | 本体智能体（Ontology-Driven Agent） | 2025H1 AI 合同 3.2 亿；爱尔眼科财务效率 8× |
| 金蝶 | 大模型 + 行业小模型双轨；灵基（Lingee）AI 原生 OS | 2025H1 AI 合同 1.5 亿；2030 年 AI 收入目标 30% |
| 浪潮 | 国资监管 / 集团管控 / 资产管理 Agent | 关联交易识别准确率 60%→95% |

## 跨行业落地图谱

| 行业 | 关键指标 |
|------|----------|
| 制造业 | 排产响应 4–6h→15min；计划外停机减 47% |
| 零售 | Walmart 供应商谈判节省 3%；毛利率 +2.3pp |
| 医疗 | 断货事件减 61%；医保审核工作量减 70% |
| 金融 | JPMorgan COiN：36 万工时→秒级；KYC 3–5 天→4 小时 |
| 能源 | 计划外停机减 47%；维护成本降 18% |
| 物流 | 运输成本降 12%；准时交付率 +19% |

**Walmart 供应商谈判 Agent**（全球零售标杆）：覆盖 2000+ 供应商，与 Pactum AI + Vertex AI 集成，谈判成功率 68–72%，周期 5–7 天→1–2 天。

**JPMorgan COiN**：年处理 1.2 万份信贷协议，提取 150 个关键属性，错误率比人工低 80%；生产环境运行 450+ Agentic AI 用例。

## 四大落地壁垒

1. **数据质量**：「任何 AI Agent 都无法弥补糟糕的数据模型」（SAP Philipp Herzig）— 主数据重复、编码不统一是硬上限
2. **Workflow 复杂度**：10–20 年深度定制积累的隐性规则无文档，成功案例多在新上线云 ERP 而非老旧本地系统
3. **Governance**：仅 21% 企业建立成熟 Agentic AI 治理模型；Oracle/Workday/SAP 将治理做成产品特性
4. **企业信任**：组织文化阻力 — 可靠性怀疑、责任归属、员工抵制；需靠生产案例逐步积累信任

Insight Partners：60% 企业 AI 计划无法扩展超过试点阶段。

## 展望

Futurum Group：73.7% 企业计划在 2025–2028 年更换核心 ERP，为 AI Native 玩家留下窗口；但大企业替换周期 3–7 年，AI Native 大规模渗透或需至 2028 年后。当前主战场：50–500 人中型企业。

核心判断：**真正改变 ERP 的不是 AI 功能，而是企业开始拥有 AI 员工。**

## 相关链接

- [[agent-skills]] — Agent Skill 规范与构建方法
- [[hermes-agent]] — 开源自我改进 Agent 与 skill 自动创建
- [[openspec]] — spec-driven 规划层框架
- [[trellis]] — 团队级 Agent 脚手架与仓库内 LLM wiki
