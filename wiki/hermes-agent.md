---
tags: [ai-agent, nous-research, open-source, tool-use, self-improving]
created: 2026-04-11
source: "https://github.com/NousResearch/hermes-agent"
---

# Hermes Agent

Nous Research 开发的开源自我改进 AI Agent，具备闭环学习能力——能从经验中创建技能、在使用中改进技能、主动持久化知识、搜索自身历史对话，并跨会话构建对用户的深度理解模型。

## 核心概念

- **闭环学习（Closed Learning Loop）**：Agent 自主管理记忆，定期触发知识持久化；完成复杂任务后自动创建 skill，且 skill 在使用中自我改进；通过 FTS5 全文搜索 + LLM 摘要实现跨会话回忆；使用 Honcho 辩证法建模用户画像
- **模型无关**：支持 Nous Portal、OpenRouter（200+ 模型）、z.ai/GLM、Kimi/Moonshot、MiniMax、OpenAI 及自定义端点，通过 `hermes model` 一键切换，无代码改动和厂商锁定
- **多平台接入**：CLI 终端 UI + 消息网关（Telegram、Discord、Slack、WhatsApp、Signal、Email），单一网关进程统一管理，支持语音备忘录转写和跨平台对话连续性
- **子代理并行**：可派生隔离的 subagent 处理并行工作流，支持通过 RPC 调用工具的 Python 脚本，将多步管道压缩为零上下文成本的 turn

## 关键特性

### 运行环境

六种终端后端：本地、Docker、SSH、Daytona、Singularity、Modal。其中 Daytona 和 Modal 提供无服务器持久化——环境空闲时休眠、按需唤醒，空闲期间几乎零成本。可以跑在 $5 VPS 上，也可以跑在 GPU 集群上。不绑定本地笔记本，可以从 Telegram 远程对话。

### Skills 系统

遵循 [agentskills.io](https://agentskills.io) 开放标准的程序性记忆系统。Agent 完成复杂任务后自动创建 skill，后续使用中持续改进。用户也可手动创建和管理 skill。

### 定时自动化

内置 cron 调度器，支持自然语言定义任务并投递到任意平台。日报、夜间备份、周审计等场景均可无人值守运行。

### 研究用途

支持批量轨迹生成、Atropos RL 环境、轨迹压缩，用于训练下一代工具调用模型。集成 Tinker-Atropos 子模块进行 RL 训练。

## 快速上手

```bash
# 安装（支持 Linux、macOS、WSL2、Android Termux）
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
source ~/.bashrc
hermes          # 开始对话
hermes model    # 选择模型
hermes setup    # 完整设置向导
hermes gateway  # 启动消息网关
```

Windows 不支持原生运行，需通过 WSL2。

## 常用命令

| 命令 | 说明 |
|------|------|
| `hermes` | 启动交互式 CLI |
| `hermes model` | 选择 LLM 提供商和模型 |
| `hermes tools` | 配置启用的工具 |
| `hermes gateway` | 启动消息网关 |
| `hermes setup` | 完整设置向导 |
| `hermes update` | 更新到最新版本 |
| `hermes doctor` | 诊断问题 |
| `/new` `/reset` | 新建对话 |
| `/model` | 切换模型 |
| `/skills` | 浏览技能 |
| `/compress` | 压缩上下文 |

## 从 OpenClaw 迁移

Hermes 的前身是 OpenClaw。通过 `hermes claw migrate` 可自动导入设置、记忆、技能和 API 密钥。设置向导会自动检测 `~/.openclaw` 目录并提供迁移选项。

## 相关链接

- [[llm-knowledge-bases]] — LLM 驱动的知识管理方法论，Hermes Agent 的记忆和 skill 系统是这一理念的实践

## 参考

- GitHub: https://github.com/NousResearch/hermes-agent
- 文档: https://hermes-agent.nousresearch.com/docs/
- Skills Hub: https://agentskills.io
- Discord: https://discord.gg/NousResearch
