# nanobot 代码模块导读指南

欢迎来到 `nanobot` 项目！这是一个极简、高效、模块化设计的个人 AI Agent 平台。为了帮助你快速上手二次开发，本指南对项目的核心模块进行了全面的拆解和代码导读。

---

## 1. 入口与配置模块 (`nanobot.py`, `cli/`, `config/`)

### **What（它是什么）**
系统的启动入口和配置管理中心。

### **Why（为什么这样设计）**
Agent 的运行需要大量配置（如 API Key、允许的通讯渠道、大模型参数等）。通过统一的配置加载器和清晰的 CLI 命令，可以方便地在不同环境（本地、Docker）启动不同的实例（如单纯的 CLI 对话、完整的 Gateway 网关）。

### **How（核心代码导读）**
- **启动入口**：[nanobot.py](file:///workspace/nanobot.py) 或 [nanobot/__main__.py](file:///workspace/nanobot/__main__.py) 是程序的入口，直接调用 CLI 模块。
- **CLI 路由**：[cli/commands.py](file:///workspace/nanobot/cli/commands.py) 包含了 `agent`（本地对话）、`gateway`（启动各渠道网关）和 `onboard`（初始化配置）等核心命令。
- **配置加载**：[config/loader.py](file:///workspace/nanobot/config/loader.py) 负责读取 `~/.nanobot/config.json`，并将其与 [config/schema.py](file:///workspace/nanobot/config/schema.py) 中基于 Pydantic 定义的数据结构进行校验和合并。

---

## 2. 消息总线模块 (`bus/`)

### **What（它是什么）**
系统内部的消息路由中枢，基于异步队列 (`asyncio.Queue`) 实现。

### **Why（为什么这样设计）**
为了实现“渠道（Channels）”和“核心大脑（AgentLoop）”的彻底解耦。Agent 大脑不需要知道当前是在 Telegram 还是 Feishu 上运行，它只需要从总线订阅消息，并将结果发布回总线。

### **How（核心代码导读）**
- **事件结构**：[bus/events.py](file:///workspace/nanobot/bus/events.py) 定义了 `InboundMessage`（用户发给 Agent 的消息）和 `OutboundMessage`（Agent 回复给用户的消息）。
- **总线实现**：[bus/queue.py](file:///workspace/nanobot/bus/queue.py) 实现了 `MessageBus` 类，提供 `publish_inbound`、`consume_inbound`、`publish_outbound` 等核心方法。

---

## 3. 通道接入模块 (`channels/`)

### **What（它是什么）**
对外对接各个 IM 平台（如 Telegram、WeChat、Feishu、Discord 等）的适配器。

### **Why（为什么这样设计）**
每个平台的 API 和交互逻辑各不相同，通过定义统一的基类接口，可以极其方便地横向扩展新的平台接入，而无需改动核心业务逻辑。

### **How（核心代码导读）**
- **统一接口**：[channels/base.py](file:///workspace/nanobot/channels/base.py) 定义了 `BaseChannel` 基类，所有具体的通道都必须实现 `start()`、`stop()` 和 `send()` 方法。
- **通道管理**：[channels/manager.py](file:///workspace/nanobot/channels/manager.py) 的 `ChannelManager` 负责读取配置，启动所有被 enabled 的通道，并监听总线的 `OutboundMessage`，将其路由调用对应通道的 `send()` 方法。

---

## 4. 大模型供应商模块 (`providers/`)

### **What（它是什么）**
大语言模型 (LLM) 的调用层。

### **Why（为什么这样设计）**
不同的模型供应商（OpenAI、Anthropic、Ollama 等）的接口规范可能略有不同。通过统一抽象，项目可以实现“一行配置切换底层模型”。

### **How（核心代码导读）**
- **模型抽象**：[providers/base.py](file:///workspace/nanobot/providers/base.py) 定义了 `LLMProvider` 基类，规定了 `chat_with_retry` 等核心模型调用接口。
- **自动注册表**：[providers/registry.py](file:///workspace/nanobot/providers/registry.py) 是一个非常巧妙的设计，它通过 `ProviderSpec` 定义了各厂商的特征，实现了按需懒加载和模型名的自动匹配路由。

---

## 5. Agent 核心大脑 (`agent/`)

### **What（它是什么）**
Agent 的核心思考、规划和执行引擎。

### **Why（为什么这样设计）**
接收用户的输入，结合历史上下文、系统提示词，交由大模型思考，并负责处理大模型返回的**工具调用 (Tool Calls)**，形成完整的闭环。

### **How（核心代码导读）**
- **中央处理器**：[agent/loop.py](file:///workspace/nanobot/agent/loop.py) 中的 `AgentLoop` 是整个系统的中枢。它从 `MessageBus` 消费消息，调用大模型，并处理响应。
- **执行器**：[agent/runner.py](file:///workspace/nanobot/agent/runner.py) 包含 `AgentRunner`，它封装了与 LLM 的具体多轮交互逻辑（包括重试、异常处理、工具并行执行）。
- **上下文构建**：[agent/context.py](file:///workspace/nanobot/agent/context.py) 的 `ContextBuilder` 负责拼装 Prompt，将 `USER.md`、`SOUL.md`、时间信息和聊天历史组合成大模型能理解的上下文格式。

---

## 6. 工具与技能模块 (`agent/tools/`, `agent/skills.py`)

### **What（它是什么）**
Agent 与外部世界交互的“手和脚”。

### **Why（为什么这样设计）**
LLM 本身只是文本生成模型，需要通过函数调用（Function Calling）来执行实际操作，如读取文件、执行脚本、搜索网络等。

### **How（核心代码导读）**
- **工具基类与注册**：[agent/tools/registry.py](file:///workspace/nanobot/agent/tools/registry.py) 和 `base.py` 管理工具的生命周期和 Schema 导出。
- **内置工具**：
  - [agent/tools/shell.py](file:///workspace/nanobot/agent/tools/shell.py)：执行 Bash 命令（支持 `bubblewrap` 沙盒隔离）。
  - [agent/tools/filesystem.py](file:///workspace/nanobot/agent/tools/filesystem.py)：文件和目录的读写。
  - [agent/tools/mcp.py](file:///workspace/nanobot/agent/tools/mcp.py)：支持 Model Context Protocol，连接外部工具服务器。
- **自定义技能**：[agent/skills.py](file:///workspace/nanobot/agent/skills.py) 解析 `skills/` 目录下的 Markdown 文件，将用户的自然语言 SOP 转换为大模型可调用的 Skill。

---

## 7. 记忆与会话模块 (`session/`, `agent/memory.py`, `agent/autocompact.py`)

### **What（它是什么）**
负责管理多轮对话的 Session，并解决大模型上下文窗口限制的长短期记忆机制。

### **Why（为什么这样设计）**
对话记录如果无限增长会导致 Token 耗尽和响应变慢。因此需要短期的**自动压缩（AutoCompact）**和长期的**核心知识提取（Dream）**。

### **How（核心代码导读）**
- **会话管理**：[session/manager.py](file:///workspace/nanobot/session/manager.py) 的 `SessionManager` 根据 `channel:chat_id` 区分不同用户的对话。
- **纯文件存储**：[agent/memory.py](file:///workspace/nanobot/agent/memory.py) 中的 `MemoryStore` 负责将历史以 JSONL 格式落盘，并管理 `MEMORY.md` 等长时记忆文件。
- **短期压缩**：`memory.py` 中的 `Consolidator` 会在 Token 预算超限时，将旧对话总结为一段摘要。
- **长期提取 (Dream)**：`memory.py` 中的 `Dream` 类在后台运行，分析历史记录并使用大模型更新 `MEMORY.md`、`SOUL.md` 和 `USER.md`，甚至沉淀出新的技能。
- **闲置压缩**：[agent/autocompact.py](file:///workspace/nanobot/agent/autocompact.py) 监控用户活跃度，在用户离开一段时间后主动触发历史压缩，降低下次唤醒的首 Token 延迟。

---

## 8. 定时与唤醒模块 (`cron/`, `heartbeat/`)

### **What（它是什么）**
系统的后台调度机制。

### **Why（为什么这样设计）**
传统的 LLM 只能“一问一答”，而真正的 Agent 应该具备“主动性”，能够在特定时间执行任务并主动给用户发消息。

### **How（核心代码导读）**
- **主动唤醒**：[heartbeat/service.py](file:///workspace/nanobot/heartbeat/service.py) 包含一个后台守护协程，默认每 30 分钟被唤醒一次。它会读取工作区的 `HEARTBEAT.md`，如果有待办任务，则伪造一条系统消息（发送者为 `system`）发给 `AgentLoop`，触发 Agent 执行任务并推送结果。
- **定时调度**：[cron/service.py](file:///workspace/nanobot/cron/service.py) 提供了更细粒度的 Cron 表达式调度能力，支持通过工具动态添加、删除周期性任务。
