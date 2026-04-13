# Summary
制定 `nanobot` 项目的系统学习计划。该项目是一个超轻量级（Ultra-Lightweight）的个人 AI Agent 平台，支持多渠道（Telegram、WeChat、Feishu 等）、多模型供应商（OpenRouter、Ollama 等）、长短期记忆管理（Dream）、工具调用（MCP、Shell、File 等）以及定时任务和被动唤醒。

# Current State Analysis
项目基于 Python 编写，核心组件结构非常清晰，主要分为：
- `agent/`: Agent 的核心循环 (`loop.py`)、上下文构建 (`context.py`)、长短期记忆 (`memory.py`、`autocompact.py`)、技能与工具 (`skills.py`、`tools/`)、子代理 (`subagent.py`)。
- `bus/` & `channels/`: 消息总线与各大聊天平台的接入层。
- `providers/`: 大语言模型 (LLM) 供应商接口的统一抽象。
- `config/` & `cli/`: 配置解析与命令行入口。
- `cron/` & `heartbeat/`: 定时与主动唤醒机制。

# Proposed Changes (学习计划与实施路径)
由于这是学习计划，具体的“变更”对应于“学习和研究的步骤”，分为以下几个阶段：

**阶段 1：项目启动与基础架构 (Foundation & Entry Points)**
- **What**: 了解项目的入口、配置系统和整体事件流。
- **Why**: 掌握 Agent 是如何启动的，以及配置参数是如何被解析并传递给核心组件的。
- **How**:
  - 阅读 `nanobot/cli/`（特别是 `commands.py` 和 `stream.py`）了解命令行入口和初始化过程。
  - 阅读 `nanobot/config/schema.py` 和 `loader.py`，掌握配置结构。
  - 阅读 `nanobot.py` 和 `__main__.py`。

**阶段 2：消息总线与渠道接入 (Message Bus & Channels)**
- **What**: 学习 nanobot 是如何接收用户消息并返回响应的。
- **Why**: 理解平台解耦的设计模式，Agent 核心不需要关心是在 Telegram 还是 Feishu 上运行。
- **How**:
  - 阅读 `nanobot/bus/queue.py` 和 `events.py`，理解 `InboundMessage` 和 `OutboundMessage` 的数据结构。
  - 阅读 `nanobot/channels/base.py` 和 `manager.py`。
  - 选择 1-2 个具体渠道（如 `telegram.py` 或 `feishu.py`）研究其接收和发送逻辑。

**阶段 3：大模型供应商接入层 (LLM Providers)**
- **What**: 学习如何统一不同大模型厂商的 API。
- **Why**: nanobot 支持非常多厂商，其注册表机制是值得学习的极简设计。
- **How**:
  - 阅读 `nanobot/providers/base.py` 理解 `LLMProvider` 抽象类。
  - 阅读 `nanobot/providers/registry.py` 了解免改动代码的注册机制。
  - 阅读具体的 Provider 实现，例如 `openai_compat_provider.py` 或 `anthropic_provider.py`。

**阶段 4：Agent 核心引擎与工具系统 (Core Engine & Tools)**
- **What**: 学习 Agent 如何思考、规划和调用工具。
- **Why**: 这是 AI Agent 的核心（大脑和手脚），决定了系统的智能程度和执行能力。
- **How**:
  - 阅读 `nanobot/agent/loop.py`（AgentLoop），这是整个系统的中央处理器。
  - 阅读 `nanobot/agent/runner.py`，了解大模型交互和重试机制。
  - 深入 `nanobot/agent/tools/` 目录，重点看 `base.py`、`registry.py`。
  - 研究具体工具：文件系统 (`filesystem.py`)、Shell (`shell.py`) 和 MCP (`mcp.py`) 的实现。

**阶段 5：上下文与记忆系统 (Context & Memory)**
- **What**: 学习 Prompt 组装与长短期记忆的实现。
- **Why**: 上下文窗口是有限的，nanobot 的 AutoCompact 和 Dream 机制非常巧妙地解决了遗忘和上下文超载的问题。
- **How**:
  - 阅读 `nanobot/agent/context.py`（Prompt Builder）。
  - 阅读 `nanobot/agent/memory.py`，深入研究 `MemoryStore`、`Consolidator`（短期记忆压缩）和 `Dream`（长期记忆提取到 Markdown 文件）。
  - 阅读 `nanobot/agent/autocompact.py` 了解闲置时的自动压缩机制。

**阶段 6：高级特性：子代理与技能、定时任务 (Advanced: Subagents, Skills, Cron)**
- **What**: 学习高阶功能的实现。
- **Why**: 了解 Agent 如何拆分任务给 Subagent 执行，如何通过 Cron 定时触发，以及如何将历史操作沉淀为 Skill。
- **How**:
  - 阅读 `nanobot/agent/subagent.py` 了解 SubagentManager。
  - 阅读 `nanobot/agent/skills.py` 了解本地技能加载机制。
  - 阅读 `nanobot/cron/service.py` 和 `nanobot/heartbeat/service.py` 学习主动唤醒机制。

# Assumptions & Decisions
- **Assumption**: 学习者已经具备一定的 Python 基础和对大语言模型 (LLM) 调用的基本认知。
- **Decision**: 采用“从外到内，再从内到外”的阅读顺序。先看输入输出（CLI、Channels、Providers），再看核心大脑（AgentLoop、Memory），最后看扩展手脚（Tools、Skills、Cron）。

# Verification steps
- **阶段性验证**: 在学习每个模块后，尝试在本地修改相关代码并运行验证（如：新增一个简单的自定义 Tool，或接入一个新的 Channel/Provider 模拟实现）。
- **终极验证**: 能够脱离原代码，手写一个包含基础通信、大模型调用和简单 Memory 机制的 Minimal Agent 脚本。