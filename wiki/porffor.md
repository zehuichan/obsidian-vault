---
tags: [javascript, typescript, compiler, aot, webassembly, runtime, open-source]
created: 2026-08-19
source: "raw/2026-08-19-porffor.md"
---

# Porffor

Porffor（威尔士语「紫色」，读 *poor-for* /ˈpɔrfɔr/）是一个从零实现的**前瞻编译（AOT）JavaScript/TypeScript 编译器**：把 JS 编成 WebAssembly、C 或原生可执行文件，运行时没有解释器、也没有 JIT。作者 [CanadaHonk](https://github.com/CanadaHonk)；MIT 许可；官网 [porffor.dev](https://porffor.dev)。截至 2026-08，GitHub 约 5.0k stars。官方自称研究项目 / alpha，**尚不适合严肃生产**。

## 核心概念

- **100% AOT**：编译期一次编完，运行期不再解释或即时编译
- **零固定 runtime**：产物只含你的代码和用到的 builtin，没有随程序附带的重型运行时
- **IR 是 Wasm**：JS → 自定义 codegen 出 Wasm 字节码 → 自研汇编器出模块；原生路径再经自研 **2c**（Wasm → C）交给 clang / gcc / zig
- **刻意不绑 LLVM / Cranelift / Binaryen**：C 作后端，便于到处编、编完还能改
- **类型当优化提示**：可解析 TS 注解（`--parse-types --opt-types`），但不做类型检查
- **Test262 驱动版本号**：如 `0.61.10` = 尚未生产就绪（大版本 0）+ 通过约 61% 官方符合性测试 + 该通过率下第 10 次构建

## 要解决什么

传统 JS 引擎（V8、SpiderMonkey）是「解释器 + JIT + 巨大运行时」：启动要预热、内存和发行体积都大。Porffor 把引擎改建成编译器，官网给出的对比（宣传数字）：

| 维度 | Porffor | 对照 |
|------|---------|------|
| 内存密度 | 1,144 apps/GB（约 0.9 MB/app） | Node 容器 28 apps/GB；self-hosted workerd 45 apps/GB |
| 发行体积 | 约 2.6 MB 单文件 | Node 137 MB、Deno 79 MB、Bun 59 MB |
| hello 原生 | ~105ms 编出 33.7KB | 捆绑整个 runtime 则大几十倍 |

卖点是**高密度部署**和**用户直接跑一个文件**，而不是再装一个 JS 运行时。

## 编译管线

```
JS/TS → Parser（默认 Acorn）→ AST → Code Generator
     → Wasm 字节码 → Optimizer → Assembler → Wasm 模块
     → 2c → C 源码 → clang/gcc/zig → 原生二进制
```

除解析器外几乎全自研。解析器可换成 Babel / Meriyah / Hermes / oxc（可选依赖）。

### 产物

| 命令 | 产物 |
|------|------|
| `porf` / `porf run` | 编译并立刻执行 |
| `porf wasm` | 独立 `.wasm`（I/O 仍依赖 Porffor 自己的 imports，不是 WASI） |
| `porf c` | C 源码 |
| `porf native` | 当前平台可执行文件 |
| `porf lambda` | AWS Lambda 包 |
| `porf profile` / `porf debug` / REPL | 分析、调试、交互 |

优化档：`-O0` 最快编译；`-O1` 默认（指令简化、import tree-shake）；`-O2` 部分求值（实验）；`-O3` 最激进。原生再加 `--cO=Ofast`（默认）等 C 编译优化。

## 类型与内存

运行时每个值是 `(value, type)` 对。数字默认 **f64**；可用实验开关 `--valtype=i32`。

内部类型里最关键的是 **ByteString**：纯 ASCII / LATIN-1 用 1 字节/字符，普通 `String` 仍是 UTF-16（2 字节/字符）。ASCII 字符串内存减半，许多 builtin 因此要写两套（`String` + `ByteString`）。`i32` 主要用于指针。

```typescript
function add(a: number, b: number): number {
  return a + b; // --opt-types 时按 number 特化，而不是「任意 +」
}
```

写 Porffor builtin 时必须显式注解：`let a: number = 1`，不能 `let a = 1`。

## 2c：Wasm → C

2c（to C）用编译器内部信息生成尽量少样板的 C，不像通用 `wasm2c`。线性内存按 64KB 页 `calloc`；函数常返回 `{ f64 value; i32 type }`。`--2cMemcpy` 用 `memcpy` 代替指针强转，更安全、可能更慢。

**原生产物没有沙箱**：可访问文件系统、网络、系统调用。不信任的代码应走 Wasm，不要走 `porf native`。可用 Zig 对生成的 C 做交叉编译。

## 与常见引擎对比

| | Porffor (AOT) | V8 / SM (JIT) | QuickJS（解释） |
|--|---------------|---------------|-----------------|
| 启动 | 立刻满速 | 需预热 | 立刻，但慢 |
| 峰值性能 | 好；原生 + 类型提示可逼近 JIT | 最好 | 差 |
| 体积 / 内存 | 极小 / 低 | 大 / 高 | 中 |
| `eval` / `Function` | 不支持 | 支持 | 支持 |
| 可预测性 | 高 | 低（看热点） | 高 |

文档给出的相对 Node 粗数量级（随负载变化很大）：解释执行 Wasm 约 2–5×；JIT Wasm 预热后接近；2c + Ofast 原生约 3–10×。CPU 密集比 I/O 密集更受益。

## 现状与缺口

- `Promise` / `await` 有已知 bug
- 不能跨作用域访问变量（参数和全局除外）
- 没有动态求值：`eval()`、`Function()` 做不到
- 大量标准库 API 未实现；Test262 通过率仍是版本号的一部分（约六成量级）
- 只用已广泛实现的 Wasm proposal：multi-value、非陷阱 float→int（必需）；bulk memory、异常处理可选；尾调用默认关

贡献门槛偏高：仓库 [AI_POLICY.md](https://github.com/CanadaHonk/porffor/blob/main/AI_POLICY.md) 要求披露 AI 使用，且 **PR 描述/评论禁止 LLM 代写**，明显 slop 会直接关。

## 安装

```bash
curl -fsSL https://porffor.dev/install.sh | sh
# 或
npm install -g porffor@latest
porf --version
```

编译器本身跑在 Node / Deno / Bun 上。出原生二进制还需要本机 C 编译器（默认 clang）。

```bash
# hello.js → 原生
porf hello.js -o hello
./hello
```

## 相关链接

- [[ilha]] — 同属 JS 生态，但方向相反：Ilha 用岛屿架构**少发 JS**；Porffor 把 JS **编没**，变成 Wasm/原生
- [[verdaccio]] — Node/npm 基础设施；Porffor 用 `npm i -g porffor` 分发，编译器宿主仍是 Node 系 runtime
- [[standard-json]] — TypeScript 工具链一侧；Porffor 把 TS 注解当成 AOT 优化提示，而不是校验层

## 参考

- GitHub: https://github.com/CanadaHonk/porffor
- 官网: https://porffor.dev
- 文档: https://canadahonk-porffor.mintlify.app/introduction
- Discord: https://discord.gg/6crs9Znx9R
