---
tags: [ai-agent, computer-use, desktop-automation, sandbox, mcp, open-source, macos, windows, linux]
created: 2026-08-03
source: "https://github.com/trycua/cua"
---

# Cua (`trycua/cua`)

Cua AI 开源的 **computer-use 2.0 基础设施**：用同一套生态覆盖后台桌面驱动（Drivers）、跨 OS Agent 沙箱（Sandbox）、评测/RL（Bench）以及 Apple Silicon 上的近原生 macOS/Linux 虚拟化（Lume）。

> 官方定位：Scale computer-use with open-source drivers, cross-OS fleets, and benchmarks · 源码 [trycua/cua](https://github.com/trycua/cua) · MIT · ~20.9k stars（剪藏时）

## 核心概念

- **四条产品线，按场景选路**：自建 Agent → Cua Sandbox；给编码 Agent 一台电脑 → Cua Drivers；评测/训练 → Cua Bench；要 macOS VM → Lume
- **后台桌面控制**：Drivers 在 macOS / Windows / Linux 上点击、打字、校验，**不抢光标与焦点**；同一 CLI + MCP，可接 Claude Code、Cursor、Codex、OpenClaw 等
- **统一沙箱 API**：`pip install cua` 后，`Sandbox` + `Image` 同一套接口覆盖 Linux 容器/VM、macOS、Windows、Android；Cloud（cua.ai）与 Local（QEMU）均可；支持 shell、截图、鼠标键盘、移动端手势
- **评测与轨迹导出**：Bench 对接 OSWorld、ScreenSpot、Windows Arena 与自定义任务，可导出 trajectory 做训练
- **Apple Silicon 虚拟化**：Lume 基于 Virtualization.Framework，提供近原生性能的 macOS/Linux VM 管理；`lumier` 提供 Docker 兼容接口

## 详细内容

### Cua Drivers（后台 computer-use）

面向「把原生桌面交给 Agent」：后台驱动应用，Linux 支持 X11 与部分 Wayland 合成器路径（对 raw background input 有明确限制）。

```sh
# macOS / Linux
/bin/bash -c "$(curl -fsSL https://cua.ai/driver/install.sh)"
```

```powershell
# Windows
irm https://cua.ai/driver/install.ps1 | iex
```

安装后按 post-install 指引操作。架构与可选 agent skill pack 见仓库 `libs/cua-driver/README.md`。

### Cua Sandbox（跨 OS Agent 沙箱）

Python 3.11+。同一 API 不绑定具体 OS/runtime：

```python
from cua import Sandbox, Image

async with Sandbox.ephemeral(Image.linux()) as sb:  # 或 .macos() .windows() .android()
    result = await sb.shell.run("echo hello")
    screenshot = await sb.screenshot()
    await sb.mouse.click(100, 200)
    await sb.keyboard.type("Hello from Cua!")
    await sb.mobile.gesture((100, 500), (100, 200))
```

| 运行时 | Linux 容器 | Linux VM | macOS | Windows | Android | BYOI (.qcow2 / .iso) |
|--------|------------|----------|-------|---------|---------|----------------------|
| Cloud (cua.ai) | ✅ | ✅ | ✅ | ✅ | ✅ | 🔜 |
| Local (QEMU) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

### Cua-Bench

```bash
git clone https://github.com/trycua/cua && cd cua/cua-bench
uv tool install -e . && cb image create linux-docker
cb run dataset datasets/cua-bench-basic --agent cua-agent --max-parallel 4
```

### Lume（macOS 虚拟化）

```bash
/bin/bash -c "$(curl -fsSL https://cua.ai/lume/install.sh)"
curl -L "$(lume ipsw | tail -n 1)" -o ~/Downloads/macos-tahoe.ipsw
lume create macos-tahoe --ipsw ~/Downloads/macos-tahoe.ipsw --unattended tahoe
lume run macos-tahoe
```

`--unattended` 离线准备 guest；`sequoia` / `tahoe` preset 会创建用户 `lume`/`lume`、开 SSH、自动登录并关掉睡眠与锁屏。Tahoe 流程已 E2E 验证；Sequoia 首次显示启动可能仍弹出 Setup Assistant 的 Accessibility 步骤（见上游 issue #2155）。

### 主要包

| 包 | 作用 |
|----|------|
| cua-driver | 跨平台后台 computer-use |
| cua-agent | computer-use 任务的 AI agent 框架 |
| cua-sandbox | 创建与控制沙箱的 SDK |
| cua-computer-server | 沙箱内 UI 交互与代码执行驱动 |
| cua-bench | 评测与 RL 环境 |
| lume / lumier | Apple Silicon VM 管理 / Docker 兼容层 |

第三方许可注意：Kasm（MIT）、OmniParser（CC-BY-4.0）；可选 `cua-agent[omni]` 含 ultralytics（AGPL-3.0）。

## 与本知识库的关系

- 与 [[agent-browser]]：后者偏浏览器 CDP/snapshot refs；Cua Drivers 偏**原生桌面后台控制**，Sandbox 则覆盖完整 OS 环境
- 与 [[opencode]]、[[grok-build]]、[[pi]]、[[hermes-agent]]：编码 Agent 可接 Cua Drivers（CLI/MCP）获得本机桌面能力，或用 Sandbox 做隔离执行环境
- 与 [[agent-skills]]：Drivers 侧提供可选 agent skill pack（见 `libs/cua-driver`）

## 相关链接

- 仓库：https://github.com/trycua/cua
- 文档：https://cua.ai/docs
- Bench：https://cuabench.ai/
- Discord：https://discord.com/invite/mVnXXpdE85
- 原始剪藏：`raw/2026-08-03-cua.md`
- 同类对照：[[agent-browser]]、[[agent-skills]]、[[hermes-agent]]、[[opencode]]
