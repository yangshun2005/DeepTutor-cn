# DeepTutor — Agent 原生架构

## 概述

DeepTutor 是一个 **Agent 原生**的智能学习伴侣，围绕两层插件模型组织——由 LLM 调用的单次 **Tools**，以及接管整个轮次的多阶段 **Capabilities**——通过三个入口暴露：CLI、WebSocket API 和 Python SDK。三者都先进入持久化的轮次应用服务，再由共享的轮次引擎将规范化上下文路由到所选能力。

## 架构

```
入口:  CLI (Typer)  |  WebSocket /ws  |  Python SDK
                    ↓                   ↓                   ↓
              ┌─────────────────────────────────────────────────┐
              │          TurnApplicationService                 │
              │   持久化、协调并重放轮次                          │
              └──────────────────────┬──────────────────────────┘
                                     ↓
              ┌─────────────────────────────────────────────────┐
              │       TurnEngine → ChatOrchestrator              │
              │   将 UnifiedContext 路由到所选 Capability         │
              │   （默认为 `chat`）                               │
              └──────────┬──────────────┬───────────────────────┘
                         │              │
              ┌──────────▼──┐  ┌────────▼──────────┐
              │ ToolRegistry │  │ CapabilityRegistry │
              │  (第 1 层)   │  │   (第 2 层)        │
              └──────────────┘  └────────────────────┘
```

`TurnApplicationService` 通过会话存储和运行时协调器持有持久化的轮次状态与重放。每次执行使用一个按轮次的 `StreamBus`；编排器发出事件，轮次运行时持久化事件，适配器再将其重放给消费者。运行时设置位于 `data/user/settings/*.json`——项目根目录的 `.env` 文件会被有意忽略。

### 第 1 层 — Tools

LLM 按需选择的单功能工具。八个用户可切换工具显示在 `/settings/tools`：

| 工具                 | 描述                                   |
| -------------------- | --------------------------------------------- |
| `brainstorm`         | 带理由的广度优先思路探索 |
| `web_search`         | 带引用的网络搜索                     |
| `paper_search`       | arXiv 预印本搜索                         |
| `zotero_search`      | 搜索已配置的 Zotero 库            |
| `reason`             | 专用的深度推理 LLM 调用             |
| `geogebra_analysis`  | 将数学图像分析为 GeoGebra 命令    |
| `imagegen`           | 生成图像                               |
| `videogen`           | 生成视频                               |

`deeptutor/tools/builtin/__init__.py` 中的 `USER_TOGGLEABLE_TOOL_NAMES` 是权威的切换列表。其他内置工具是**上下文门控**的或由能力拥有的：`CONFIGURABLE_BUILTIN_TOOL_NAMES` 声明上下文门控表面，而 `deeptutor/agents/_shared/tool_composition.py` 拥有挂载规则（`ToolMountFlags`）和始终可用的工作区工具。示例包括 `rag`、记忆和笔记本工具、`read_skill`、延迟的 MCP/CLI 工具、`exec`、`ask_user` 以及掌握度导航。`--tool` 从用户可切换白名单中选择；它不会绕过上下文或能力门控。

### 第 2 层 — Capabilities

拥有整个轮次的多阶段流水线：

| 能力       | 阶段                                                |
| ---------------- | ----------------------------------------------------- |
| `chat`           | exploring → responding（单 Agent 循环，默认） |
| `ask_questions`  | responding（聊天循环强制通过 `ask_user`）      |
| `deep_solve`     | responding（聊天循环 + 求解规划工具）         |
| `deep_question`  | ideation → generation                                 |
| `deep_research`  | rephrasing → decomposing → researching → reporting    |
| `visualize`      | analyzing → generating → reviewing（SVG / Chart.js / Mermaid / HTML；或通过 `render_type` 路由到 Manim 子阶段） |
| `math_animator`  | concept_analysis → concept_design → code_generation → code_retry → summary → render_output |
| `mastery_path`   | responding（引导式学习——聊天循环 + 掌握度工具，按主题类型门控） |
| `immersive_reading` | responding（基于文档或时间戳的媒体阅读循环） |
| `video_lesson`   | code_generation → render_check → render_output（Remotion 课程在浏览器中实时播放，带 TTS 旁白） |

所有能力都汇聚到 `deeptutor/agents/_shared/capability_result.py` 中的 `emit_capability_result()`，因此每个轮次都发出相同的信封（响应载荷 + 来自 `UsageTracker` 的 `cost_summary`）。状态文案和提示词通过 `capabilities/prompts/{en,zh}/<name>.yaml` 进行 i18n。

## CLI 用法

```bash
# Install from the current dev source checkout
python -m pip install -r requirements/full.txt
python -m pip install -e . --no-deps

# Alternative in 1.7.0: choose one in a separate wheel environment
python -m pip install "deeptutor[full]"  # Full app, CLI + Web/API
python -m pip install deeptutor         # Core CLI/SDK (chat + ask_questions)
python -m pip install deeptutor-cli     # Compatibility alias for core

# Run any capability
deeptutor run chat "Explain Fourier transform"
deeptutor run deep_solve "Solve x^2=4" --kb my-kb
deeptutor run visualize "Animate sine wave" --config render_mode=manim_video

# Interactive REPL
deeptutor chat
# (inside the REPL: /regenerate or /retry re-runs the last user message)

# Partners (IM-connected companions)
deeptutor partner list

# Knowledge bases, memory, server
deeptutor kb list
deeptutor kb create my-kb --doc textbook.pdf
deeptutor memory show
deeptutor serve --port 8001       # API server only
deeptutor start                   # backend + frontend together
```

核心之外的示例需要其所属模块；使用 `deeptutor module list` 查看已安装/已启用/就绪状态，使用 `deeptutor plugin list` 发现已注册的能力和工具。源码 Web 启动还需要 Node.js 22 LTS，并在 `web/` 中执行 `npm ci --legacy-peer-deps`。仅使用 CLI 的源码用户可跳过前端设置，但仍需完整的 Python 依赖。凭据和验证请遵循 [Agent Setup](docs-for-dev/AGENT_SETUP.md)。

## 关键文件

| 路径                                       | 用途                              |
| ------------------------------------------ | ------------------------------------ |
| `deeptutor/runtime/orchestrator.py`        | `ChatOrchestrator` — 统一入口   |
| `deeptutor/runtime/launcher.py`            | 后端 + 前端生命周期 / 端口发现 |
| `deeptutor/runtime/registry/`              | 工具 + 能力注册表         |
| `deeptutor/runtime/bootstrap/builtin_capabilities.py` | 内置能力类路径 |
| `deeptutor/services/config/runtime_settings.py` | JSON 设置 + 进程环境变量覆盖 |
| `deeptutor/services/subagent/`             | 本地/远程 Agent 连接器；在 `registry.py` 中注册每个后端，在 `models.py` 中注册其模型选项（Grok CLI 使用原生 `streaming-json`） |
| `deeptutor/core/stream.py`, `deeptutor/runtime/stream_bus.py` | StreamEvent 协议 + 异步扇出 |
| `deeptutor/core/tool_protocol.py`          | `BaseTool` + `ToolDefinition`         |
| `deeptutor/core/capability_protocol.py`    | `TurnCapability` + `CapabilityManifest` |
| `deeptutor/core/context.py`                | `UnifiedContext` 数据类            |
| `deeptutor/tools/builtin/__init__.py`      | 所有内置工具包装器           |
| `deeptutor/capabilities/`                  | 内置能力实现  |
| `deeptutor/app/`                           | `DeepTutorApp` SDK 门面 + 持久化应用服务 |
| `deeptutor_cli/main.py`                    | Typer CLI 入口点                |
| `deeptutor/api/routers/unified_ws.py`      | 统一 WebSocket 端点           |

## 依赖层

1.7.0 布局是模块化的；1.6.15 及更早的包使用之前的布局。核心拥有 CLI、SDK 和共享运行时；功能 wheel 必须与其版本匹配。`deeptutor-cli` 是仅元数据的兼容别名。开发时，使用源码或[本地 wheel](docs-for-dev/MODULES.md)。

模块清单拥有依赖组；生成的 requirements 文件支持源码、Docker 和 CI 安装。`pyproject.toml` 将额外模板存储在 `tool.deeptutor.optional-dependencies` 下；`build_metadata.py` 解析同版本 wheel 固定。源码检出暴露所有模块，并通过 `requirements/full.txt` 安装外部依赖，随后执行 `pip install -e . --no-deps`。使用源码辅助脚本安装可选额外项，而不是在检出上安装功能 wheel：

```bash
python scripts/install_extras.py partners       # Partner channel SDKs
python scripts/install_extras.py matrix         # Matrix channel without E2EE/libolm
python scripts/install_extras.py matrix-e2e     # Matrix with end-to-end encryption
python scripts/install_extras.py math-animator  # Manim; also needs system LaTeX/ffmpeg
python scripts/install_extras.py dev            # Test / lint tooling
python scripts/install_extras.py all            # Full + Manim + dev dependencies
```

更改模块前请停止 DeepTutor。源码支持 `module enable/disable`；wheel 环境还支持 `module install/uninstall`。升级旧版 `deeptutor-cli` 安装需要先卸载它，再安装新包，并保留相同的 `DEEPTUTOR_HOME`；参见设置指南。使用 `python -m pip check`、`deeptutor module list` 和设置就绪检查进行验证。