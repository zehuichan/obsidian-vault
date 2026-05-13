---
tags: [agent-skills, design, ai-harness, anti-pattern, design-system, frontend, open-source]
created: 2026-05-13
source: "https://impeccable.style/"
---

# Impeccable

由 Paul Bakaus（Renaissance Geek）开发的 **Agent Skill 包**，把"设计素养"装进任何 AI 编码助手（Cursor、Claude Code、Codex CLI、Gemini CLI、VS Code Copilot、Antigravity、Kiro、OpenCode、Pi、Trae、Rovo Dev）。它把 23 条命令、25–27 条反模式检测规则、PRODUCT.md + DESIGN.md 共享上下文、以及一个浏览器内的 Live Mode 编为一体，自称是"Anthropic 官方 `frontend-design` skill 的升级版"。当前版本 v3.0.7，Apache 2.0。

> Great design prompts require design vocabulary. Most people don't have it. Impeccable teaches your AI deep design knowledge and gives you 23 commands to steer the result.

## 核心理念

- **设计提示需要设计词汇**：用户没有词汇就无法精准指挥 AI；Impeccable 把这些词汇编码为命令（`/typeset`、`/colorize`、`/animate`、`/polish`、`/audit`、`/critique`…），让普通工程师用专业语言驱动 AI 出图
- **品牌（brand）与产品（product）是两套语法**：营销页和仪表盘的设计准则是矛盾的，所有命令按 register 切换内部参考资料——这是 Impeccable 区别于 Anthropic 原版 `frontend-design` 的关键
- **共享上下文文件优于反复提示**：`PRODUCT.md`（受众、品牌人格、反参考、register）与 `DESIGN.md`（颜色、字体、组件，Google Stitch 规范）由命令生成并在每次调用时加载，输出不再退化为"通用 SaaS 风格"
- **AI 已经学坏了同一批 slop**：所有大模型都收敛到紫色渐变、嵌套卡片、Inter 字体、低对比度标签等可枚举模式，必须用**确定性规则**而非 LLM 自评来拦截
- **设计该在写代码的地方发生**：放弃 Figma → 交付的瀑布流，直接在 Claude Code / Cursor / Codex 内对真实代码库迭代

这是把 [[agent-skills]] 中"Skill 不是普通 Prompt"的设计模式推到极致的范例：一个 `/impeccable` 命名空间下挂 23 条子命令、约 40 个引用文件、一条 CLI 工具、一个 Chrome 扩展。

## 三层架构（Foundation / Language / Case）

### 01 Foundation — 七份参考文件

每次 prompt 前先加载七份学科参考：**typography · color · motion · spatial · interaction · responsive · UX writing**。运行 `/impeccable teach` 一次即给项目锚定设计基线，后续所有命令都从此读取。

这对应 [[agent-skills]] 中描述的 **L2 指令层 + L3 资源层渐进式加载**：命令本体短小，深度知识下沉到按需读取的 `references/`。

### 02 Language — 23 条命令

每条命令对应一个设计学科或一种操作意图：

- **修复型**：`/polish`（综合优化）、`/typeset`（字体）、`/layout`（布局间距）、`/colorize`（色彩）、`/animate`（动效）
- **质检型**：`/audit`（5 维评分 + P0-P3 严重度）、`/critique`（按 Nielsen 十大可用性启发 + 人格化子 Agent 平行评审）、`/detect`（确定性反模式扫描）
- **风格调节**：`/bolder`、`/quieter`、`/distill`、`/delight`、`/overdrive`（极致视觉效果）
- **创建型**：`/shape`（结构化需求访谈生成品牌工具包）、`/craft`（把生成的 mock 转成真实代码站点）、`/teach`、`/document`、`/extract`、`/harden`（生产化加固）
- **元命令**：`/pin`（把子命令固定为顶层快捷方式）、`/unpin`、`/skills update`、`/live`（见下）

输入 `/impeccable` 即可看到 autocomplete 全列表。`/impeccable pin audit` 会写一个轻量重定向 Skill，让 `/audit` 重新作为顶层命令工作。

### 03 Case — 设计哲学的落地

- **`/impeccable teach` 产出 PRODUCT.md**：受众、品牌人格、反参考、register（brand/product）。每条命令读取后再生成
- **`/impeccable document` 产出 DESIGN.md**：扫描 tokens、组件、渲染结果，写成 [Google Stitch 六段式规范](https://stitch.dev)，可被任何 DESIGN.md-aware 工具复用
- **`/impeccable shape` + `/impeccable craft`**：先生成品牌工具包图像（GPT Image 2 / Nano Banana Pro / Imagen 4 Ultra），再让 Codex/Claude 把图像作为"视觉契约"实现为响应式代码（v3.0.4 起把 mock 缺失元素当作 blocking defect）
- **`/impeccable polish` 是生产代码的对齐器**：扫描 `tokens.css`、`tailwind.config.ts`、组件目录，匹配现有系统而不发明新系统

## 反模式引擎与"Gallery of Shame"

Impeccable 给"AI 设计垃圾"命名并量化。**25 条浏览器规则 + 27 条 CLI 规则**，确定性、无 LLM、无 API key、无评分波动。

被点名的模式包括：

- **Purple Gradients**（紫色渐变 hero）
- **Cardocalypse**（卡片里再套卡片）
- **Side-Tab Cards**（左侧厚边框装饰条）
- **Inter Everywhere**（字体单一化）
- **Italic-serif display heroes**（Fraunces/Recoleta/Newsreader/Playfair/Cormorant/Tiempos 的大号斜体衬线 hero——v3.0.7 新增，被认定为 2025 末到 2026 初 AI 营销页的结构性指纹）
- **Hero eyebrow chips**（hero h1 上方那块全大写疏散字距的胶囊标签，含 999px 圆角变体）
- **Overused fonts**（除 Inter 外新增 Fraunces、Geist、Mona Sans、Plus Jakarta Sans、Space Grotesk、Recoleta、Instrument Sans 黑名单；对 Vercel、Next.js、GitHub 自家域名豁免）
- **Gradient text headings**、**bad-contrast labels**、**ai-color-palette**、**template layouts**

三种分发面：

- **Skill 内调用**：`/impeccable audit`、`/impeccable polish` 直接在 AI 里检查
- **CLI**：`npx impeccable detect src/`，27 条规则，`--json` + `--fast` + exit code，用于 PR 检查与 CI 门禁
- **Chrome 扩展**：在 localhost、staging、生产、竞品页面上一键扫描，叠加在原页面 DOM 上高亮

## 双 Register：brand vs product

Impeccable 最锋利的分歧点。Anthropic 自家的 `frontend-design` skill（以及大多数同类 skill）把所有设计场景塞进同一套词汇，但：

- **brand register**——设计 _就是_ 产品（落地页、营销站、作品集、编辑内容）
- **product register**——设计 _服务于_ 产品（仪表盘、工具型 app、内部系统）

两套场景的字号节奏、留白、动效、文案语气、信息密度互相矛盾。`typeset`、`animate`、`colorize`、`bolder`、`quieter`、`layout`、`delight` 等命令按 register 切换内部决策表。Register 由任务推断并在 `PRODUCT.md` 中确认。

## Live Mode（Alpha）

`/impeccable live` 启动后：

1. 在浏览器里**点选**任意元素
2. 写注释或随手画一笔，按 Go
3. AI 生成**三个生产级变体**，通过你框架的 HMR 实时切换
4. 选 Accept，变体**直接写回源文件**
5. `live status` / `live resume` / `live complete` 处理崩溃恢复（v3.0.7 加入持久会话日志）

工程实现亮点（散见于 v3.0.1–v3.0.5 changelog）：

- **CSP 兼容**：检测到 HTML 中的 CSP meta 标签时，启动时静默追加 live-server 源到 `script-src`/`connect-src`，停止时逐字节还原
- **JSX/TSX wrap → preview → accept → carbonize 循环**：wrap 用单 JSX-slot 子节点而不是三兄弟，能干净穿越 `return (...)`、`.map(...)`、Radix 的 `asChild`
- **重复同类兄弟消歧**：picker 把 `textContent` 作为 `--text` 参数传入，必要时报 `element_ambiguous` 而不是默选第一个
- **Modal host 兼容**：在 Radix Dialog、Headless UI、vaul 这类锁 `pointer-events: none` 的 portal 内强制 chrome 自身 `pointer-events: auto`
- **身份保留为默认**：v3.0.6 起变体只在已有视觉身份内探索不同表达轴，除非 PRODUCT.md 反参考显式否定当前界面或用户明确要求偏移

## 安装与运行

```
$ npx skills add pbakaus/impeccable          # 主装法，自动识别 12 种 harness
$ /plugin marketplace add pbakaus/impeccable # Claude Code plugin
$ npm i -g impeccable                        # 全局 CLI
$ npx impeccable skills update               # 升级 skill 定义
```

- 项目根的 `PRODUCT.md` 与 `DESIGN.md` 在升级时**不会被覆盖**
- Cursor 需开启 Nightly + Settings → Rules 中的 Agent Skills
- Codex 的 skills 不在 `/` picker 里，要用 `/skills` 或 `$` 打开

## 演进路径与启发

时间线上的几个关键 commit：

- **v1.0（2026-02）**：17 条命令的 frontend-design 升级版
- **v2.0（2026-04-08）**：改名为 impeccable；**对 15 个设计 brief 做 with/without skill 的盲评 A/B**，最显著的单点改动是"**让模型先说出前三个直觉再否决它们**"再选字体/调色板；加入 27 条确定性反模式 + CLI + Chrome 扩展
- **v3.0（2026-04-10）**：Live Mode、PRODUCT.md/DESIGN.md、brand/product register、18 条独立 skill 收编为 1 个带 23 子命令的 skill（也就是 `/impeccable` 命名空间）
- **v3.0.7（2026-05-04）**：italic-serif hero / eyebrow chip 检测、recoverable live sessions、参考文件去除冗余 scaffolding（删掉 419 个 em-dash 和 12 个老命令的 "[Verb] [object] to [outcome]" 套话），目的是减少模型每次加载的上下文与重复

值得记下的实践经验：

- **Reflex-reject 表**：与其让模型自由选择，不如先列出它最容易选的字体/配色/版式套路并强制否决——把"AI 训练数据偏差"显式建模为反模式
- **二阶反射检查**：v3.0.6 引入"两个高度"的 reflex check——既拦截"从品类直接选主题/配色"的一阶反射，也拦截"从品类 + 反参考导出某个美学家族（如 editorial-typographic）"的二阶反射
- **mock 作为视觉契约**：v3.0.4 起 `/impeccable craft` 在 build 前要列出 mock fidelity inventory，缺失任一显式元素都是 blocking defect——把"AI 把生成图当装饰"的滑坡显式封死
- **autonomous-agent 加固**：Codex 类 harness 倾向把自己的总结当作 brief 就开始写文件，所以 craft 现在需要显式 `shape=pass` 门禁

## 与既有文章的关系

- [[agent-skills]] — Impeccable 是 "Skill 即可移植行为模块" 概念的高完成度范例：一个命名空间、23 子命令、按 register 分支的引用资料、可重定向的 `/pin` 机制、有评估闭环（v2.0 的 15 brief A/B）的开发流程
- [[open-design]] — 同样是"把 AI 编码助手转化为设计引擎"的开源工作流，但路径不同：Open Design 走 **31 skills + 72 design systems** 的横向铺开（多 skill、多品牌系统、BYOK Web 面板），Impeccable 走 **1 skill × 23 命令 × 双 register** 的纵向打磨（更像"设计语言学的 IDE 扩展"）。两者都是 Apache 2.0、都支持 12 种主流 harness、都把 `DESIGN.md` 当作可移植资产

## 风险与局限

- **Live Mode 仍是 Alpha**：CSP、modal host、TSX round-trip 都是过去两个月才陆续修好的问题，对复杂渲染管线（SSR + 流式、Server Components）仍有未知边界
- **反模式表会过时**：italic-serif hero 这种规则本质是"对训练数据流行风格的负反馈"，需要持续跟进；Vercel/Next.js/GitHub 的自家域名豁免也说明规则会与某些品牌正在做的事直接冲突
- **23 个命令的认知负担**：作者用 `/impeccable pin` 提供顶层快捷方式来缓解，但学会哪条命令对应哪种意图本身仍是一笔学习成本
- **依赖 Skill 触发机制**：在 Codex 的 `/` picker 之外、在 Cursor Nightly 之外，触发路径并不统一，需要逐个 harness 验证

## 参考

- 官网：[impeccable.style](https://impeccable.style/)
- GitHub：[pbakaus/impeccable](https://github.com/pbakaus/impeccable)
- npm：`impeccable`
- 推特：`@impeccable_ai`
- 作者：Paul Bakaus（Renaissance Geek）
- 相关：[[agent-skills]]、[[open-design]]
