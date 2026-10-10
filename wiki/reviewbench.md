---
tags: [ai-agent, code-review, benchmark, evaluation, github, leaderboard]
created: 2026-10-10
source: "raw/2026-10-10-reviewbench.md"
---

# ReviewBench

ReviewBench 是 GitHub 开源的**离线 AI 代码审查基准**：在真实 pull request 上，用人工复核过的 golden findings 衡量审查 Agent 能否找出有用问题、同时压住误报，并按严重程度和类别拆开看结果。

> 官方定位：an open, reproducible benchmark for evaluating AI code review systems on real-world pull requests · 仓库 [review-bench/ReviewBench](https://github.com/review-bench/ReviewBench) · 榜单 [review-bench.ai](https://review-bench.ai) · MIT · 剪藏时 41 stars / 9 forks

方法论全文见 [docs/METHODOLOGY.md](https://github.com/review-bench/ReviewBench/blob/main/docs/METHODOLOGY.md)。

## 核心概念

- **Finding**：挂在变更代码某一文件、某一行范围上的原子审查意见，至少包含路径、行范围、自然语言说明和生产者标识
- **Golden set**：每个 PR 上、语料构建阶段标好 TP/FP 并经人工复核的参考集，是评分的地面真相
- **True positive**：资深审查者会期望作者在合并前处理的意见——事实正确、可对照变更代码核验、属于这次审查的范围、且可执行
- **False positive**：不值得处理的意见，包括事实错误、无法核验、超出本次 PR 范围、重复、低于项目审查门槛的风格吹毛求疵，以及按建议改了反而变差的意见
- **Grounded 指标**：只用 golden set 标签。未匹配上的候选 finding 不进分子也不进分母
- **Augmented 指标**：未匹配的候选 finding 再交给同一分类器标 TP/FP，把「golden set 没收录、但审查者自己发现的问题」算进去
- **官方裁判**：榜单终评使用 Claude Sonnet 5。调参和本地打分用你自己的模型和密钥

它针对的是现有代码审查基准的两极：手工精选的少量典型 bug（精确但覆盖不了真实审查行为），以及「后续 commit 是否改到同一处」这类自动代理（覆盖广但标签噪）。ReviewBench 要同时度量**发现质量**和**发现覆盖**。

## 语料

完整集 **219** 个 PR，来自 **187** 个仓库；单一仓库最多 10 个 PR（4.6%）。测试集是其中 **25** 个 PR 的代表性样本，覆盖每种 finding 类别和严重程度，用于适配器验证，**测试集分数不能当榜单分数**。每个源仓库在 `review-bench` 组织下有镜像 `review-bench/<owner>_<repo>`，保存该任务的 base 与 head commit；上游被删或改写后基准仍可跑。

候选 Agent 看到的 diff 固定在「审查反馈给出时」的 head，作者随后为回应评论而推的 commit 只作为抽取 finding 的证据，不进入待审 diff。

### 语言

| 语言 | 测试集 PR | 占比 | 全集 PR | 占比 |
|------|----------:|-----:|--------:|-----:|
| TypeScript | 5 | 20.0% | 68 | 31.1% |
| Python | 4 | 16.0% | 41 | 18.7% |
| C# | 3 | 12.0% | 25 | 11.4% |
| Go | 3 | 12.0% | 19 | 8.7% |
| JavaScript | 1 | 4.0% | 15 | 6.8% |
| 其他 | 9 | 36.0% | 51 | 23.3% |
| **合计** | **25** | **100%** | **219** | **100%** |

测试集的「其他」为 Rust、Java、Jupyter Notebook、Kotlin、PHP、Ruby、Shell、Swift。

### 变更规模（增删行数）

| 行数 | 测试集 | 全集 |
|------|-------:|-----:|
| ≤50 | 3（12.0%） | 17（7.8%） |
| 51–200 | 5（20.0%） | 40（18.3%） |
| 201–500 | 5（20.0%） | 44（20.1%） |
| 501–1,000 | 5（20.0%） | 40（18.3%） |
| >1,000 | 7（28.0%） | 78（35.6%） |

### Finding 分布（测试集，共 352 条）

| 严重程度 | 条数 | 占比 |
|----------|-----:|-----:|
| High | 36 | 10.2% |
| Medium | 134 | 38.1% |
| Low | 182 | 51.7% |

类别：Correctness 133、Reliability 58、Maintainability 44、Testing 35、Security 27、Documentation 21、Performance 12、API architecture 12、Accessibility 10。

全集 PR 类型由标题和描述推断（非正式人工标签）：Feature 79（36.1%）、Bug fix 59（26.9%）、Documentation 15、Performance 14、Refactor 12、Other 40。

语料目前不接收新的 PR 提交。

## 标注怎么来的

每个 PR 的候选 finding 来自多类生产者，避免 golden set 偏向单一来源：

1. **人工审查评论**：行锚点评论直接抽出
2. **推断的作者动作**：作者后续 commit 像是在修一个从未被写成评论的问题；推断必须能被前后 diff 支撑
3. **确定性工具**：linter、类型检查、静态分析，只取打在变更行上的规则违反
4. **LLM 审查 Agent**：保留原始行锚点和原文

归一化到同一 schema 后，由分类器标注。TP 要同时满足真（可核验）、相关（改了会让 PR 更好）、属于这次审查。Scope 单独记录为 `introduced-by-pr` / `exacerbated-by-pr` / `pre-existing` / `unrelated`，预存在问题仍可以是 TP。每条 finding 还有 severity（high / medium / low）、单一主类别、难度、所需上下文、可执行性。

初始标签由 Claude Sonnet 4.6 分类器打出，模型弃用后改为 Claude Sonnet 5。分类器输出经人工审计：

- **TP/FP 一致率 96.6%**。每条分歧都人工核对；人工曾误标的情况包括「报告的 bug 已被现有 guard 挡住」。分类器的真实漏判集中在：忽略已有缓解措施，或接受前提不成立的 finding。人工改掉了 47 条被误标为 TP 的 finding
- **Severity**：三档上相差不超过一档 98.7%，精确一致 62.9%。人类把问题标得更严重的次数大约是标得更轻的九倍，golden set 偏保守
- **Category 精确一致 79.7%**。残留分歧主要在 correctness 与 reliability 的边界（例如吞掉的异常算结果错误还是容错缺口）

跨生产者的重复 finding 用与评测相同的匹配过程去重，避免 recall 分母被灌水。Golden set 不假定穷尽：没被任何生产者提出的有效问题仍然存在，这是 augmented 指标存在的原因。标签可通过公开争议流程改判。

分类器 prompt 公开在 [`scripts/classifier/prompts.ts`](https://github.com/review-bench/ReviewBench/blob/main/scripts/classifier/prompts.ts)。

## 评分

对每个 PR：Agent 在固定的 base/head 上产出候选 findings → LLM matcher 做语义匹配 → 未匹配项用同一分类器标 TP/FP → 算出四项指标，并可按 severity / category 分层。榜单跑三轮，公布三轮算术平均，并保留每轮分数以便审计方差。耗时（`review_ms`）单独报告，不进入 precision、recall 或 \(F_\beta\)。

匹配按文件分组，只比较同一文件内的候选与 golden。两条意见描述同一底层问题即匹配，即使措辞和行号不同；同一位置描述不同问题则不匹配。对应关系是多对多。计分时：

- **Recall**：至少一个候选 raw-match 到某条 golden，该 golden 即被覆盖，多条候选命中同一条只计一次
- **Precision**：每个候选最多认领一条尚未被认领的 golden（按 golden 顺序的第一条），继承其标签。其余对应仍可给后面的候选认领，也可继续计入 recall

| 指标 | 含义 |
|------|------|
| Grounded precision | 认领了 golden 的候选里，有多少对应已知 TP：\(\|M_{TP}\| / \|M\|\) |
| Grounded recall | 已知 TP 里，Agent 覆盖了多少 |
| Augmented precision | 全部候选里，有多少是有用的（匹配到的 TP + 分类器新判的 TP），分母是全部候选 |
| Augmented recall | 把 Agent 新发现的 TP 同时加进分子和分母 |

Augmented recall 的分母随 Agent 变化：新发现很多分类器认可的 TP 时，即使 grounded recall 相同，augmented recall 也会被抬高。横向比较 Agent 时以 grounded recall 为准；augmented recall 和新 TP 数量用来看它能不能提出语料之外的问题。

聚合同时报 **macro**（每个 PR 等权，看典型 PR）和 **micro**（先合并计数再算比率，看典型 finding）。榜单可按严重程度、类别，以及偏 precision 或偏 recall 的 \(F_\beta\)（\(\beta<1\) 偏精度，\(\beta>1\) 偏召回）实时重算，过滤发生在服务端，不是对已聚合分数做客户端加权。

## 如何上榜

适配器很薄：读一个 PR，写一个 findings 文件。文档称某个开源审查器大约 90 行，主要是字段映射。参考实现是 [`examples/codex-cli`](https://github.com/review-bench/ReviewBench/tree/main/examples/codex-cli)。

1. 按 [AGENT_CONTRACT.md](https://github.com/review-bench/ReviewBench/blob/main/AGENT_CONTRACT.md) 把 Agent 打进容器
2. 推到 GHCR，记下 digest
3. 用 `scripts/try-agent.sh` 在测试集上验证容器能跑完并产出合法 findings（这一步不打分）
4. 在 [review-bench.ai/submit](https://review-bench.ai/submit) 用 GitHub 登录注册。网站代开 onboarding PR，在 `agents/` 写入符合 schema 的 manifest，维护者合并。联系邮箱不进 PR

之后：测试集试跑（25 PR，逐条看适配器输出）→ 你在自己的算力上对全集调参 → 选定配置后做 **三轮全集终评**。只有经这个自助流程发起的终评才能上榜。没有月度次数上限。你选的是送评的配置，终评本身会重新测，不能指定某一次本地跑分。

本地裁判（不跑容器、不上榜）：

```sh
npm ci
export OPENAI_API_KEY="<your key>"
npm run judge -- \
  --candidate ./my-agent-findings \
  --provider openai \
  --model <your-model-id> \
  --output ./scoring/smoke.json \
  --limit 1
```

费用：调参、测试跑和反复打全集，Agent 推理和裁判推理都用你的密钥。终评时 ReviewBench 承担裁判费用，Agent 仍用你的密钥，因为模型和配置本身就是被测对象。凭证只在网站录入，进 Azure Key Vault，直接注入容器，跑完即清除；manifest 只声明变量名和文件路径。建议使用可随时吊销、带消费上限的专用密钥，不要把密钥打进镜像。

## 局限

- Golden set 是有限生产者的并集，大家共同漏掉的问题仍然不可见
- TP/FP 边界带有工作组的口味；分类器错误会进入 grounded 和 augmented 两边。缓解方式是把 severity、category、scope 分开记录，版本化分类器，并公布与人工标签的一致率
- 匹配依赖 LLM matcher：漏匹配会把 grounded 记分变成 augmented 记分，错匹配会把没真正找到的问题算成命中
- 每个 PR 固定在一个 snapshot 上评分
- 离线标签是「真人会不会采纳这条意见」的代理，应拿线上信号（评论反应、解决率、后续 commit）回标

## 相关链接

- [[opencode]] — 开源 AI 编码 Agent，可作为被评测的审查器宿主
- [[grok-build]] — 终端编码 Agent，含 headless 运行，适合包进审查容器
- [[lean4-skills]] — 面向 Agent 的 prove / review / golf 循环，审查在这里是证明工作流的一环
- [[codegraph]] — 本地代码知识图谱，给审查 Agent 提供调用链和符号上下文
- [[agent-skills]] — Agent Skill 规范；ReviewBench 的适配器是另一类「薄封装」接口
- [[ponytail]] — 约束 Agent 少写代码的规则集，其 benchmark 单独保证安全维度不被简化
