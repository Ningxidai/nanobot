# Summary
为 `nanobot` 项目的各个核心模块制定代码解释文档的生成计划，帮助用户更好地理解项目架构并快速上手二次开发。

# Current State Analysis
`nanobot` 是一个超轻量级的个人 AI Agent 平台，代码总行数精简，但功能全面。项目采用高度解耦的模块化设计：
- 核心引擎 (`agent/`)：处理 LLM 交互、工具调用、上下文管理和长短记忆。
- 通信层 (`bus/`, `channels/`)：通过事件总线隔离了具体的聊天平台（如 Telegram、Feishu）与 Agent 核心。
- 模型接入层 (`providers/`)：统一抽象了不同大模型的 API。
- 支撑模块 (`cli/`, `config/`, `cron/`, `session/`)：负责应用启动、配置解析、定时任务和会话管理。

# Proposed Changes
计划在工作区生成一份详细的模块代码导读文档：`/workspace/.trae/documents/nanobot_modules_guide.md`。该文档将包含以下模块的详细代码解释（What/Why/How）：

1. **入口与配置模块 (`nanobot.py`, `cli/`, `config/`)**
   - **What**: 项目的启动入口和配置加载。
   - **Why**: 了解 Agent 是如何被初始化、如何读取 `~/.nanobot/config.json` 的。
   - **How**: 解释 `cli/commands.py` 中的命令路由，以及 `config/loader.py` 的配置合并逻辑。

2. **消息总线模块 (`bus/`)**
   - **What**: 系统内部的消息路由中枢。
   - **Why**: 实现通道层和核心引擎的解耦。
   - **How**: 解释 `events.py` 中的 `InboundMessage` / `OutboundMessage` 数据类，以及 `queue.py` 中 `MessageBus` 的发布/订阅实现。

3. **通道接入模块 (`channels/`)**
   - **What**: 对接外部 IM 平台（Telegram, WeChat 等）。
   - **Why**: 接收用户输入并将其转化为标准内部消息，同时将 Agent 响应发回平台。
   - **How**: 解释 `base.py` 中 `BaseChannel` 的接口定义，以及 `manager.py` 如何管理所有启用的通道。

4. **大模型供应商模块 (`providers/`)**
   - **What**: 统一不同大模型 API 的调用接口。
   - **Why**: 方便随时切换模型（如 OpenAI, Anthropic, Ollama）而无需修改核心逻辑。
   - **How**: 解释 `base.py` 中的 `LLMProvider` 抽象类，以及 `registry.py` 如何实现免修改代码的模型注册。

5. **Agent 核心大脑 (`agent/`)**
   - **What**: 系统的思考和执行中枢。
   - **Why**: 负责组装 Prompt、调用 LLM、执行工具。
   - **How**:
     - `loop.py`: 解释 `AgentLoop` 如何循环处理消息。
     - `runner.py`: 解释 `AgentRunner` 如何处理工具调用和重试。
     - `context.py`: 解释 `ContextBuilder` 如何拼接系统提示、工具信息和历史记录。

6. **工具与技能模块 (`agent/tools/`, `agent/skills.py`)**
   - **What**: Agent 与外部世界交互的手脚。
   - **Why**: 提供执行 Shell、读写文件、MCP 协议等能力。
   - **How**: 解释 `registry.py` 中的工具注册机制，以及 `skills.py` 如何将特定目录下的 prompt 转换为工具。

7. **记忆与会话模块 (`session/`, `agent/memory.py`, `agent/autocompact.py`)**
   - **What**: 管理多轮对话和长短期记忆。
   - **Why**: 防止上下文超限，实现 Agent 的长期记忆。
   - **How**: 解释 `SessionManager` 的会话存储，`Consolidator` 的短期历史压缩，以及 `Dream` 机制的长时记忆提取。

8. **定时与唤醒模块 (`cron/`, `heartbeat/`)**
   - **What**: 允许 Agent 在没有用户输入时主动执行任务。
   - **Why**: 实现类似日常总结、定时提醒的助理功能。
   - **How**: 解释 `CronService` 的调度逻辑，以及 `HEARTBEAT.md` 驱动的周期性任务。

# Assumptions & Decisions
- **Assumption**: 用户希望快速建立对代码库的整体认知，以便进行二次开发或定制。
- **Decision**: 将提供一份总览性的导读文档，提炼出每个目录下的关键类和函数，而不是逐行解释。这有助于用户先把握骨架，再深入细节。

# Verification steps
1. 检查生成的 `/workspace/.trae/documents/nanobot_modules_guide.md` 文件是否包含了上述所有模块。
2. 确认文档中的代码引用（如类名、文件路径）是否准确对应当前代码库。
3. 用户阅读导读文档，确认是否达到了“更好先入手项目”的目的。