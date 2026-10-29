# DeepTutor v1.4.0-beta 发布说明

**发布日期：** 2026.05.21

v1.4.0-beta 是 agent-native 重写以来最大的版本。它在现有能力之上叠加了端到端 **Auto Mode**，交付了带专属工作台的**三层记忆子系统（L1/L2/L3）**，将 **Deep Research / Deep Solve / Question** 重建在与 Chat 相同的 agentic 引擎上，围绕会话累积式来源清单重新架构了 **chat 能力 + LlamaIndex RAG 流水线**，统一了**能力基础设施与 i18n**，将 Animator 菜单合并进 **Visualize**，并重组了**设置、环境与本地启动器**。此外还新增了多个聊天工具（`ask_user`、`web_fetch`、`write_note`、`list_notebook`、`github_query`）、删除聊天轮次流程、测验追问聊天以及 GeoGebra 查看器。

## 亮点

### Auto Mode —— Agentic 能力路由器
新增的 `auto` 能力位于现有模式之上，为每个请求选择合适模式，而非强制用户预先选择。

- **三阶段 agent 循环** —— `ANALYZING`（单次 LLM 调用，以思考流形式输出）→ `DELEGATING`（最多 `max_iterations` 次路由调用，发出 `delegate_to_<cap>` 工具调用或原子工具调用）→ `SYNTHESIZING`（最终内联回答，由循环透传或由收尾 LLM 调用组装）。
- **路由到真实能力** —— `deep_solve`、`deep_question`、`deep_research`、`math_animator`、`visualize` 以及聊天级原子工具（`web_search`、`web_fetch`、`rag` 等）都位于同一路由器之后，LLM 可在单轮内混合检索与完整子能力运行。
- **有界重试与配额** —— 路由 LLM 错误、单次委派失败、参数校验反馈各有独立重试预算；可配置的 `max_same_capability_calls` 配额防止循环在单一模式上空转。
- **干净的对话历史** —— 子能力事件经 `forward_events` 垫片流转，为每个内容事件打上 `call_id` 标签，因此对话轮次运行时过滤器在保存的历史中只保留 Auto 自身的最终综合。子运行仍实时流式推送到 UI。
- **`answer_now` 快速路径** —— 用户要求"立即回答"时，流水线跳过分析与委派，直接生成内联回复。

### 三层记忆子系统（Memory v2）
原先扁平的记忆页面被替换为结构化三层存储，配有显式整合流水线与专属工作台。

- **L1 / L2 / L3 布局** —— L1 捕获原始运行轨迹，L2 保存规范化文档记录，L3 保存各界面（chat、notebook、book、TutorBot）的精选槽位。每用户路径经 `PathService` 流转，多用户部署保持隔离。
- **Consolidator 流水线** —— 模块化 `consolidator/` 模块（chunker、guards、parse、references、runs、modes、line-doc、meta）将运行轨迹转为带稳定 id、层间引用与快照历史的版本化行式文档。
- **Memory Workbench UI** —— 新增 `/memory` 路由（`graph`、`l1`、`l2`、`l3`、`resolve`），作为独立页面提供工作台、hub、图查看器、运行面板与归档状态横幅。可复用的 `MemorySection` 组件嵌入原记忆面板所在位置。
- **一等聊天工具** —— `read_memory` 与 `write_memory` 作为 agent 工具暴露（带 i18n 提示），chat / Auto 可在单轮内召回与更新记忆，无需单独保存步骤。
- **设置集成** —— Memory 在 `/settings/memory` 下拥有独立页面，含运行控制、模式开关与存储状态。

### Deep Research、Deep Solve 与 Question 迁移至 Agentic 引擎
三条多 agent 流水线已重写为共享 agentic-engine 原语之上的编排器，删除了数百个定制提示文件与各 agent 类。

- **Deep Research → `agents/research/pipeline.py`** —— 四个阶段（`Rephrase`、`Decompose`、`Research blocks`、`Reporting`）实现为带标签步骤（`THINK` / `TOOL` / `APPEND` / `OUTLINE` / `SECTION` / `FINISH`）。动态主题队列与 `CitationManager` 保留；新增 `APPEND` 标签让研究块无需离开循环即可向队列添加后续主题。`ask_user` v2 驱动最多三轮改写，支持多问题卡片。
- **Deep Solve → `agents/solve/pipeline.py`** —— `Pre-retrieve`（仅知识库）、`Plan`、`Solve`（每步 `THINK` / `TOOL` / `FINISH` / `REPLAN` 循环，含 solve 到 plan 的回边）以及最终 `Synthesize` 步骤。每步的 `FINISH` 流入下一步的提示上下文，使答案读起来是一段连续叙述。
- **Question / Quiz** —— 协调器 + 流水线取代旧的 `generator` / `idea_agent` / `models` 模块；旧提示目录已完全移除。
- **三者均删除各自模式的旧 `agents/` 与 `prompts/` 目录**，只留一个流水线文件与共享的带标签步骤提示。

### Chat 能力与 LlamaIndex RAG 重构
agentic chat 流水线围绕会话累积式"Attached Sources"清单与更简洁的 LlamaIndex 流水线重建。

- **分支隔离的来源清单** —— `services/session/source_inventory.py` 物化当前分支祖先链上附加的所有来源。当前轮次的新来源显示完整预览；历史来源显示单行，含 id、名称、类型、大小及首次出现的轮次序号。LLM 调用 `read_source(id)` 按需展开全文。兄弟分支之间不会泄漏来源。
- **LlamaIndex 流水线拆分** —— 专用 `config.py`、`ingestion.py`、`retrievers.py`、`document_loader.py` 取代此前的单体流水线模块。存储与 v1.3 版本化索引保持向后兼容。
- **精简 agentic chat 提示** —— `agentic_chat.yaml`（EN/ZH）已重写以匹配新工具面与来源清单契约；旧的并行工具提示脚手架已移除。
- **内置工具注册表** —— `tools/builtin/__init__.py` 是聊天挂载工具、提示提示与参数增强包装器的唯一注册处。

### 能力基础设施统一
每个能力现在都经过同一共享信封、同一状态 i18n 加载器与同一成本追踪面。

- **`emit_capability_result` 辅助函数** —— 每个能力通过一个辅助函数发出最终结果，一致地填充结果信封（标签、摘要、载荷、渲染提示）与尾部用量追踪器总计。
- **`StatusI18n`** —— 能力状态文案位于 `capabilities/prompts/{en,zh}/<name>.yaml`，经共享 `StatusI18n` 访问器加载。流水线中硬编码的英文状态字符串已移除。
- **`UsageTracker` 成本面** —— token 用量与成本经每次能力运行一个追踪器记录，暴露到结果信封，并显示在新的 `/settings/capabilities` 管理页（实时列表、默认值、按能力覆盖开关）。
- **移除已弃用的 `main.yaml` 键** —— 旧 `main.yaml` 能力副本已删除，改用按能力的提示文件。

### Visualize：Animator 合并为单一能力
独立的 Animator 菜单已合并进 Visualize，用户只需选择一次可视化，系统自行选择渲染器。

- **`render_type` 判别器** —— `AnalysisAgent` 从六种渲染类型中选择：`svg`、`chartjs`、`mermaid`、`html`（文本输出，三阶段流水线）或 `manim_video` / `manim_image`（Manim 子进程流水线）。结果信封携带 `render_type`，前端据此委派给正确的查看器。
- **单一侧边栏入口** —— 旧 `Animator` 菜单项已移除；用户现通过 `Visualize` 处理静态图表与 Manim 视频。全屏查看器 / 配置面板处理所有渲染类型。

### 新增聊天工具
- **`ask_user`** —— 将 1–3 个结构化问题打包为单个载荷，暂停当前轮次直到用户回答。前端渲染卡片，让用户浏览问题并批量提交答案；流水线以答案作为工具结果恢复该轮次。Deep Research 的 Rephrase 阶段使用，chat / Auto 也可用。
- **`web_fetch`** —— URL 抓取，含可读内容提取、严格的 scheme / 私有 IP / 大小防护（预检与重定向后均应用），输出超限时加 `…[truncated]` 标记。
- **`write_note`** —— 取代旧的 `save_to_notebook` 工具。两种模式：`append` 创建新记录（默认正文为渲染后的转录，可选 agent 撰写正文），`edit` 按 `record_id` 更新现有记录。
- **`list_notebook`** —— 对当前用户笔记本与记录的只读索引 / 下钻列表。仅在用户确有笔记本时挂载，因此空运行在构造上不可能发生。
- **`github_query`** —— 只读 `gh` CLI 包装器，覆盖 `pr`、`issue`、`run`、`repo` 以及仅 GET 的 `api` 回退。工具面无法触达任何变更动词。`gh` 未安装时返回干净的"工具不可用"结果。

### Chat 界面功能
- **删除聊天轮次**（#443）—— 消息项现携带稳定 `id`，会话 API 暴露 `deleteMessage`，chat reducer 新增 `DELETE_TURN` 动作，409 vs 404 检查拒绝删除仍在运行的轮次。删除前解析乐观临时 id，避免 UI 行孤立。
- **测验追问聊天输入框** —— `FollowupChatComposer` 与 `QuizFollowupContext` 让用户直接从测验题发起聊天线程。输入框复用主 `ChatComposer`（外观、@space 选择器、知识库选择器、附件、LLM 选择器），但发送经专用追问控制器路由。配套 `quiz-judge.ts` 辅助函数支持内联评判追问答案。
- **测验 UI 打磨** —— 测验答案文本域可垂直调整大小（#478）；题目内容将单个换行规范化为 Markdown 段落（#441）。
- **GeoGebra 查看器** —— `Geogebra.tsx`、`GeogebraOpenCTA.tsx` 与 `GeogebraTabContext` 新增 GeoGebra applet 渲染器（经官方 GGB applet 脚本加载），几何 / 代数片段可在聊天答案旁内联打开。

### 多用户数据隔离
v1.3.x 引入多用户后的若干回归与缺口在专项修复中解决（#474、#465）。

- **认证与中间件解耦** —— 多用户身份解析不再依赖全局中间件状态，修复了特定路由顺序下导致跨用户数据泄漏的变基回归。
- **旧会话管理器路径捕获** —— 旧会话管理器正确继承当前用户作用域，文件路径落在每用户工作区内而非共享默认位置。
- **前端全面使用 `apiFetch`** —— 每个已认证客户端调用现经 `apiFetch()`，认证头一致附加。
- **SSL 绕过清扫** —— `DISABLE_SSL_VERIFY` 现覆盖 codex provider 与四个在 v1.3.10 后仍缺失的嵌入适配器。

### 环境设置、安装器与本地启动器
安装 + 启动流程已重写，移除 `.env` 解析迷宫，使 `deeptutor start` / `deeptutor init` 成为一等命令。

- **`runtime_settings.py`** —— 系统 / 认证 / 启动设置现位于一个带显式默认值的类型化模块（`backend_port`、`frontend_port`、`cors_origins`、`disable_ssl_verify`、`chat_attachment_dir` 等），JSON 存储于 `data/user/settings/`。280+ 行的旧 `env_store.py` 与两个 `.env.example` 文件已删除。
- **`runtime/launcher.py`** —— 单一异步启动器，掌管后端 + 前端生命周期、端口发现、就绪探测与清理。生成 `web/.env.local`，使 Next.js 前端始终获取解析后的后端端口。
- **`deeptutor/runtime/banner.py`** —— `deeptutor start` 与 `deeptutor init` 共享的本地化启动横幅；从界面设置读取语言偏好，使横幅与 UI 语言一致。
- **`init_wizard.py`** —— 交互式 `deeptutor init` 向导，含 provider 菜单、API key 环境变量自动检测、实时 `GET {base_url}/models` 拉取、精选回退列表，以及保存前可选的连通性探测。
- **`model_catalog.py` 精简** —— 目录文件缩减约 400 行，按 provider 的样板代码移入 `provider_registry` 与适配器模块。

### 设置 UI 重组
单一 `/settings` 页面已拆分为聚焦标签页。

- **新路由** —— `/settings/appearance`、`/settings/capabilities`、`/settings/embedding`、`/settings/llm`、`/settings/mcp`、`/settings/memory`、`/settings/search`、`/settings/status`、`/settings/tools`，共享布局与条目索引。
- **Tools 页** —— 列出所有可挂载到聊天的工具，显示可用性（如 `github_query` 的 `gh`），并提供按工具开关。
- **Capabilities 页** —— 将新的 `UsageTracker` 成本面与上述按能力默认值及覆盖开关配对。

### Zulip 频道集成
TutorBot Zulip 频道（v1.3.9 加入）获得后续修复清扫与自订阅功能（#480）。

- **为 @提及 自动订阅频道** —— Bot 可自行订阅任何 @提及 它的频道，从而真正在主题中收到消息。已订阅频道警告降级为 info 级，启动日志不再误导性地标记成功路径。
- **支持所有提及标志类型** —— `mentioned`、`wildcard_mentioned`、`topic_wildcard_mentioned`、`stream_wildcard_mentioned` 均触发 bot，修复频道 `@` 提及静默问题。
- **附件发送修复** —— 重发附件不再将 Zulip 上传路径当作本地文件，上传辅助函数不再因 `'str' object has no attribute 'name'` 崩溃，缺失的路由元数据从 `_recipient_map` 重建，消除 `Message must have recipients` 错误。
- **进度消息去重** —— 内部 `_tool_hint` 进度事件从频道发送中过滤，用户不再看到重复的"tool starting…"行。
- **测试覆盖** —— 新增附件上传 + 发送恢复与频道订阅行为的单元测试。

## 测试

- Auto 流水线、委派、Schema 与 `auto` 能力面的新测试 —— 1100+ 行新覆盖，含端到端 agent 循环行为。
- 新记忆子系统完整测试覆盖 —— chunker、consolidator、document、ids、line-doc、merge、meta settings、modes、ops、references、runs、store。
- `ask_user`、`github_query`、`list_notebook`、`web_fetch`、`write_note` 的按工具单元测试，以及 ask-user UI 状态辅助函数。
- 按 agentic-engine 标签（`THINK` / `TOOL` / `APPEND` / `FINISH` / …）重构 chat / research / solve / question 流水线测试。
- 新增会话 / 来源清单测试，覆盖分支隔离与累积清单行为。
- 前端测试覆盖 message-branches 辅助函数、版本面与 ask-user 状态机。

## 升级说明

- **设置文件迁移** —— 首次启动会将任何基于 `.env` 的设置迁移到 `data/user/settings/` 下的新 JSON 文件。旧 `env_store` 垫片已移除；若你外部脚本化写入 `.env`，请改为指向 `runtime_settings.py` 或 `/settings` API。
- **推荐使用 `deeptutor start` 启动** —— `start_web.py` / `start_tour.py` 仍可用，但现为新 `runtime/launcher.py` 的薄包装。在新机器上运行一次 `deeptutor init` 以初始化 provider 与凭据。
- **Animator 菜单用户** —— 请改用 **Visualize**。用户要求视频 / 动画时，该能力现自动选择 Manim；现有 Manim 渲染记录不受影响。
- **记忆数据迁移** —— 旧单块记忆格式由 consolidator 在首次访问时读取，并写回为 L2 / L3 记录。无需手动步骤；旧快照仍保留在磁盘上。
- **能力作者** —— 经 `capabilities/_shared.emit_capability_result` 发出结果，并将状态文案放入 `capabilities/prompts/{en,zh}/<name>.yaml`。硬编码英文状态字符串将无法通过审查。
- **Beta 范围** —— 本版本交付大量新界面（Auto、Memory v2、设置拆分）。在生产环境请固定到 `v1.4.0-beta` 直至 GA 版本；欢迎针对任何新模块提交 bug 报告。

**完整变更日志**：https://github.com/HKUDS/DeepTutor/compare/v1.3.10...v1.4.0-beta