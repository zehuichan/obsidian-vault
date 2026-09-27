---
tags: [json-schema, dynamic-form, architecture, postgresql, cqrs, outbox, multi-tenant]
created: 2026-09-27
source: "raw/2026-09-27-json-schema-dynamic-data-model.md"
---

# 企业级动态数据模型

把 JSON Schema 当作**源语言**而不是运行时：编译成可执行、可缓存、可灰度、可观测的动态数据模型平台，用「写事实、建投影、做分析」扛住日均亿级提交，而不是在每个请求里现解析 Schema、现改表。

> 动态表单的终极解法，不是「把字段做成可配置」，而是「把数据模型做成可编译、可执行、可治理、可演进的平台能力」。

## 核心概念

- **四个核心对象**：模型定义、模型运行时、模型数据、模型事件。动态表单只是入口
- **Schema 是源，IR 是运行时**：语法校验 → 语义扩展校验 → AST/IR → 校验器、渲染树、索引计划、安全策略、版本路由
- **主链路最小闭环**：幂等 → 加载已编译 Schema → 同步校验 → 写事实 + Outbox。统计、搜索、通知、分析全部后移
- **JSONB 存事实，投影承担查询**：高频过滤走物化列，检索走 ES/OpenSearch，分析走 ClickHouse/Doris
- **版本化缓存**：key 带版本号（`model:tenant:modelKey:v15`），灰度期新旧版本并存，禁止用「最新版本」组织缓存

## 详细内容

### 为什么 Demo 会在企业场景崩掉

Demo 路径是：前端按 JSON Schema 渲染、后端顺手校验、库里存一份 JSON。企业场景立刻升级：租户一天几十个表单且 Schema 高频变更；单表单几百字段含联动与跨字段校验；日提交过亿、热点活动几分钟打满写链路；要求不停机发版、灰度、历史版本兼容；风控、搜索、分析、通知、审计都要消费同一份动态数据；租户字段各不相同，平台仍要统一治理。

线索采集的典型退化：第一周只要手机号、城市、产品、隐私协议；第二周加年龄、预算、是否有车、渠道；第三周企业版变成企业名称、统一社会信用代码、行业、开票信息、联系人岗位。传统改页面 / 改 DTO / 加列 / 联调 / 发版会被业务拖垮。引入 JSON Schema 之后的第二波事故是：每次请求现解析导致 RT 飙高；多实例缓存不一致（A 节点 v7、B 节点 v6）；旧数据按新 Schema 校验失败；热门表单缓存同时失效打爆数据库；JSONB 能存但不能按手机号、渠道、活动批次查（全表扫描）；风控、搜索、分析没有统一出口。

真正的五个挑战：定义如何稳定演进、运行时如何高效执行、数据如何高并发写入与查询、版本如何灰度/兼容/迁移、能力如何被搜索/风控/分析复用。

### 四个对象

| 对象 | 内容 | 关注点 |
|------|------|--------|
| 模型定义 | 字段、类型、规则、版本、UI 元数据、权限元数据 | 定义治理，不是用户提交 |
| 模型运行时 | 编译结果、校验器、渲染树、索引计划、路由规则 | 编译一次、多次复用 |
| 模型数据 | payload、物化列、索引、快照、审计 | 写入可靠、可追溯 |
| 模型事件 | 发布、失效、迁移、投递、聚合、回放 | 风控/通知/分析/搜索的统一扩展点 |

打通之后：业务不用等研发改表；前后端围绕同一元模型；平台统一治理多租户、多版本、多环境；写、查、分析各自演进。

### JSON Schema 只做源语言

一份定义同时承载四类信息：结构（字段名、类型、嵌套）、校验（`required` / `pattern` / `enum` / `minimum`）、UI（控件、文案、占位符）、治理（索引、加密、脱敏、权限）。示例扩展：

- `x-ui`：`component`、`title`、`placeholder`
- `x-index`：`mode`（如 `btree`）、`materialize`
- `x-security`：`pii`、`encrypt`、`mask`、`readRoles`

生产环境不能在每个请求里临时读 Schema、临时解析、临时组装规则。正确做法是编译成平台自己的中间表示，避免被某个校验库绑死（文中示例用 `networknt/json-schema-validator`，换库时业务层不感知）：

```java
public record DynamicModelIR(
        String modelKey,
        long version,
        Map<String, FieldDef> fields,
        List<ConstraintDef> constraints,
        List<IndexPlan> indexPlans,
        SecurityPlan securityPlan,
        UiRenderTree renderTree) {}
```

编译链路：

```text
Schema JSON -> 语法校验 -> 语义扩展校验 -> AST / IR
            -> 后端校验器 -> 前端渲染树 -> 索引计划
            -> 安全策略 -> 版本路由元数据
```

至少六类产物：`ValidatorRuntime`、`UiRenderTree`、`IndexPlan`、`SecurityPlan`、`RulePlan`（跨字段、跨步骤、条件联动）、`VersionPlan`（兼容、默认值补齐、迁移）。

### 分层与模块

```text
api / application / domain / infrastructure / worker / admin
```

- `api`：表单查询、提交、回放
- `application`：编排提交、查询、发布
- `domain`：模型定义、版本规则、约束、提交聚合、事件
- `infrastructure`：PostgreSQL、Redis、Kafka、ES、对象存储
- `worker`：异步迁移、投影、索引同步、统计聚合
- `admin`：设计器、发布台、灰度控制台、版本审计

五个模块：`schema-registry`（定义存储、发布、编译、缓存与失效）、`submission-service`（高并发提交、幂等、同步校验、写事实）、`projection-worker`（查询投影、搜索索引、统计快照）、`query-service`（筛选、分页、导出、审计读取）、`schema-admin`（可视化设计、预览、回滚、灰度）。

架构一句话：**写事实、建投影、做分析，不要混在一条主链路里。**

```text
Schema Admin UI
    -> Schema Registry (publish / compile)
         |-- cache warm --> Redis L2
         |-- schema-change event --> Kafka / Pulsar
API/BFF -> Submission Service (validate / save)
              |-- PostgreSQL JSONB
              |-- Outbox
         -> Projection Worker -> ES / ClickHouse / OLAP
```

### 领域对象

**ModelDefinition**（可发布定义）：`modelKey`、`version`、`status`、`schemaJson`、`extensions`、`compatibleFromVersion`、`createdBy`、`publishedAt`。

**ModelSubmission**（交易事实）：`submissionId`、`tenantId`、`modelKey`、`schemaVersion`、`requestId`、`submitToken`、`payload`、`materializedColumns`、`traceId`、`createdAt`。

**ModelProjection**：运营筛选、搜索索引、统计快照、审计日志。同一份提交事实派生多份读模型。

**ModelEvent**：`SchemaPublishedEvent`、`SchemaRollbackEvent`、`SubmissionAcceptedEvent`、`ProjectionRefreshEvent`、`SchemaMigrationRequestedEvent`。

### 存储：JSONB + 投影，而不是 EAV 或动态宽表

| 方案 | 优点 | 缺点 | 建议 |
|------|------|------|------|
| EAV | 极灵活 | 查询差、行爆炸、聚合差 | 不作为主方案 |
| 动态宽表 | 查询快 | DDL 频繁、扩展差 | 只适合稳定字段 |
| JSONB + 投影 | 灵活、可查询、可演进 | 要治理索引和投影 | 最推荐 |

三张核心表：

- `model_definition`：`tenant_id + model_key + version` 唯一；`schema_json JSONB`、`schema_hash`、`compatible_from`、`status`；查找索引 `(tenant_id, model_key, status, version DESC)`
- `model_submission`：`payload JSONB` + 物化列（如 `materialized_mobile/city/channel`）；幂等唯一键 `(tenant_id, model_key, submit_token)`；手机号用部分索引 `WHERE materialized_mobile IS NOT NULL`
- `outbox_event`：`aggregate_type/id`、`event_type`、`event_key`、`payload`、`status` 默认 `NEW`；投递索引 `(status, created_at)`

300 字段的表单不要给 200 个字段建表达式索引：写入变慢、索引膨胀、vacuum 变贵、热更新难治理。字段分层：

1. 事实字段：只进 JSONB，不建索引
2. 高频过滤：物化列或表达式索引
3. 检索字段：同步到 ES/OpenSearch
4. 分析字段：异步写入 ClickHouse / Doris / Iceberg

### 写链路：主链路只留四步

错误写法把 Schema 查询、全量校验、写入、统计、风控、通知、搜索索引、AI 打标全塞进一次同步提交。后果是事务长、锁久、线程阻塞、外部依赖放大 RT、任一失败拖垮主流程。

正确闭环：

```text
接收请求 -> 鉴权 / 限流 / 反作弊 -> 幂等校验
        -> 加载 CompiledSchema -> 同步校验
        -> 补齐默认值 / 生成物化列
        -> 写 model_submission + outbox_event -> 返回成功
Outbox Relay -> Kafka -> Projection / Search / Stat / Audit Worker
```

Outbox 比「事务里直接发 Kafka」稳：避免「MQ 成功、库回滚」或「库成功、MQ 失败」。把业务一致性从分布式两阶段提交降成单库事务 + 可重试投递。

幂等不能靠前端防连点。文中 `IdempotencyService` 用 Redis `SET NX`：key `idem:{tenant}:{model}:{submitToken}`，先写 `PROCESSING`（10 分钟），成功后改 `DONE:{submissionId}`（24 小时）。重复 token 直接拒绝。

提交服务的生产边界：幂等在最前；同步只做必要动作；敏感字段经 `SecurityPlan` 统一加密；`IndexPlan.materialize` 的字段物化；事件只写 outbox。

### 读链路：查询不要反向污染写模型

三类读模型：在线查询（运营筛选、分页、导出）、检索（全文、模糊、组合条件）、分析（报表、漏斗、分群、画像）。

- PostgreSQL JSONB + 物化列：事实与部分精确查询
- Elasticsearch / OpenSearch：复杂搜索
- ClickHouse / Doris：OLAP

不要把运营查询透传成 `payload->>'field'`。平台查询 DSL 只暴露字段、算子和分页，由元数据决定走物化列、表达式索引、ES 还是分析引擎：

```json
{
  "modelKey": "lead-capture",
  "filters": [
    {"field": "mobile", "op": "eq", "value": "13800000000"},
    {"field": "budgetRange", "op": "in", "value": ["10-30w", "30-50w"]},
    {"field": "createdAt", "op": "between", "value": ["2026-08-01", "2026-08-20"]}
  ],
  "pageNo": 1,
  "pageSize": 20
}
```

### 版本、灰度与迁移

状态机：`DRAFT -> TESTING -> BETA -> PUBLISHED -> ARCHIVED`。

- `DRAFT`：编辑中，不对外
- `TESTING`：仅内部联调
- `BETA`：按租户、比例、渠道放量
- `PUBLISHED`：正式版本
- `ARCHIVED`：不再接新流量，仍可读取

灰度不只是发布/回滚两个按钮：按租户、环境、渠道、用户比例、请求头指定版本。例如 VIP 租户走 v15、抖音渠道走 v14、其余走 v13。

管理端应拦住高风险变更：已发布字段不许直接改类型；删除必填走破坏性变更流程；新增必填必须有默认值或只对新流量生效；枚举收缩要校验历史数据；索引计划变更要审批。事故往往来自「模型变更不受控」，不是代码 bug。

迁移是一等公民：管理端发任务 → 生成迁移计划 → Worker 分批扫描 → 幂等补默认值或做结构转换 → 写审计 → 校验完成后再切流量。这是从动态表单进化到数据模型平台的分水岭。

### 三级缓存与热更新

运行时对象很重（原始 JSON、编译后的 validator、渲染树、索引计划、安全元数据），不能每次回源再编译。

```text
L1: Caffeine 本地缓存
L2: Redis 集中缓存
L3: PostgreSQL / Config Store
```

本地缓存接绝大多数读；Redis 共享原始定义或编译快照；DB 是唯一真实源。缓存编译结果或编译原料，不缓存临时中间态。热点模型主动预热，不等首请求冷编译。文中本地缓存 `maximumSize(20_000)`、`expireAfterWrite(15min)` 并 `recordStats()`；Redis 存 schema JSON，TTL 6 小时；L1 存 `CompiledSchema`。

热更新：管理端发布 → 写 `model_definition` → 同事务写 `SchemaPublishedEvent` 到 outbox → Relay 投 Kafka topic `schema-change` → 各实例失效本地缓存和 Redis → 异步预热 → 新流量按路由命中新版本。比「改 Schema 就重启」成熟。

一致性要点：

- 缓存 key 带版本：`dyn-model:{tenant}:{modelKey}:v{version}`
- 本地只缓存 versioned key，不用「最新版本」这个概念
- 热点用 single-flight 或分布式锁，避免击穿（Redis 删掉后全实例同时回源）
- 灰度期新旧版本并存，不强行同一时刻全量切换

### 多租户与字段级安全

多租户不是加一列 `tenant_id`。四层隔离：定义隔离、数据隔离、缓存 key 带租户、限流隔离（大租户不能拖垮全平台）。缓存、索引、消息路由少带一个租户维度就会出隐蔽串读。

敏感字段（手机号、身份证、邮箱、企业信息）的规则写进 `x-security`，平台统一处理：入库前加密、按角色脱敏、导出裁剪、审计敏感字段访问。不要让每个业务系统自己判断该不该脱敏。

### 前端消费渲染协议，不消费原始 Schema

把原始 JSON Schema 直接丢给组件库，短期快，长期有三个问题：UI 被 Schema 标准能力卡住；复杂联动、布局、异步字典难做；换组件库成本高。

后端维持 JSON Schema 为源；编译期生成 `UiRenderTree`；前端只解释自己的渲染协议（`name` / `component` / `label` / `required` / `props` / `options`）。前端职责收成：协议解释器、条件联动执行器、表单状态、异步字典与按需加载。同一协议便于跨端复用。

### 营销活动平台的三阶段

1. **静态表单**：每活动一页，加字段就改代码。交付慢、复用差、历史版本乱。
2. **JSON Schema 动态表单**：动态渲染 + 动态校验 + JSON 存储。大部分字段变更不用发版，但查询差、版本乱、多实例缓存不一致、高峰编译和回源贵。
3. **动态数据模型平台**：编译与版本治理、三级缓存与事件失效、JSONB + 物化列、Outbox + 异步投影、多租户与字段安全、运营查询 DSL。变更大多在管理端完成；主链路短；搜索、统计、导出围绕统一模型扩展；同一底座承载问卷、留资、审批补录、企业资料采集。

### 可观测、降级与部署

没有观测就没有容量。四组指标：

| 域 | 看什么 |
|----|--------|
| 提交 | QPS、成功率、P95/P99、幂等冲突率、校验失败率、单租户占比 |
| 模型 | 编译耗时、L1/Redis 命中率、热点模型数、版本失效率 |
| 存储 | 连接池、慢 SQL、JSONB 查询耗时、写入 TPS、分区膨胀 |
| 事件 | Outbox 积压、MQ lag、投影延迟、重试次数、死信数 |

每次提交带上 `traceId`、`tenantId`、`modelKey`、`schemaVersion`、`submissionId`、`requestId`，才能串起网关、提交日志、数据库、Kafka、投影消费。

高峰降级：单租户限流、热点模型限流、查询与提交隔离、导出异步化、非核心投影延迟构建、风控超时自动降级。稳定是「失败时仍然可控」。

Pod 吃 CPU（编译、序列化、校验）、内存（compiled schema 缓存）和到 Redis/DB/MQ 的网络。部署注意：JVM 堆够用但不过量（示例 `-Xms1g -Xmx1g -XX:+UseG1GC`）；HPA 不只看 CPU，还看请求量和 RT；新 Pod 先预热热点模型；readiness 成功前不接流量。示例资源 request `500m/1Gi`，limit `2 CPU / 2Gi`，readiness `/actuator/health/readiness`。

压测不要只看 QPS，要同时看 P95/P99 是否劣化、连接是否打满、Redis RT、缓存命中率、MQ lag、单租户热点是否拖垮全局。复盘要回答瓶颈在哪一层、是 CPU/锁/连接池/IO 还是下游、这次优化解决了哪个结构性问题。

### 落地流程与五个坑

**发布**：需求 → 模型设计 → 平台规则校验 → 测试环境预览 → 联调 → 灰度 → 看指标 → 正式发布 → 归档。

**提交**：客户端 → 网关鉴权限流 → 幂等 → 版本路由 → 同步校验 → 敏感字段处理 → 写事实 → 写 Outbox → 异步投影 → 搜索/统计/通知/分析。

**高风险变更**：改类型必须评审；删字段必须有历史兼容方案；新增必填必须有默认值或新流量策略；扩大索引必须评估写性能；新增敏感字段必须补加密与脱敏。

**校验失败率飙升时**：冻结新发布 → 切回稳定版本 → 对比新旧 Schema → 查默认值缺失或枚举收缩 → 回放失败样本 → 修复后重新灰度。

| 坑 | 现象 | 解法 |
|----|------|------|
| Schema 当数据库模型 | 定义与存储耦合，查询一来就补丁改表 | Schema 只做源；存储、索引、投影走编译产物 |
| 规则全同步 | 主链路长，外部抖动拖垮提交 | 同步只做结构校验和核心业务校验 |
| 缓存 key 不带版本 | 新旧切换串数据 | 缓存、投影、渲染协议全部 versioned |
| 所有查询打在 JSONB 上 | 在线查询变慢，索引变重 | 事实表兜底；高价值查询走物化列、ES 或分析库 |
| 做了动态字段、没做安全 | 手机号/身份证裸奔，导出全量泄漏 | 安全元数据进 Schema；平台统一加密、脱敏、裁剪、审计 |

### 四个推进阶段

1. **定义统一**：JSON Schema 规范 + `x-ui` / `x-index` / `x-security` + 发布与版本
2. **运行时稳定**：编译器、本地缓存 + Redis、提交幂等与 outbox
3. **查询与分析解耦**：高价值字段物化、搜索/分析异步投影、查询 DSL
4. **产品化**：可视化管理台、灰度、迁移任务、字段安全、多租户与审计

走到第四阶段，承载的就不再是一个表单功能，而是营销留资、问卷、企业资料补录、审批扩展、风控采样、运营配置都能复用的底座。

七点结论：JSON Schema 适合做源语言，生产必须编译；动态表单本质是动态数据模型；主链路只做幂等、校验、写事实、发事件；JSONB 是起点，查询/搜索/分析靠投影；多租户、版本、灰度、字段安全是必修课；缓存一致性、事件投递、可观测和限流决定能不能扛住高峰；成熟形态是企业级动态数据模型平台，不是动态表单服务。

## 相关链接

- 原文：https://mp.weixin.qq.com/s/ou-pTgZSpMIEiuhPmdgFzg
- 原始剪藏：`raw/2026-09-27-json-schema-dynamic-data-model.md`
- [[standard-json]]：把 Zod/Valibot 等运行时 schema **转成** JSON Schema；本文把 JSON Schema **编译成** 平台运行时，方向相反、可接在同一条链上
- [[openapi-ts-request]]：从 OpenAPI 生成 JSON Schemas 与请求代码，适合契约侧；本文关注契约落地后的高并发提交与投影
