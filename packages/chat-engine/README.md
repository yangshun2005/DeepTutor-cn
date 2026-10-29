# 实验性 TypeScript 聊天引擎

本包在 `typescript-native` 配置下负责聊天/伙伴**代理循环、Chat Completions 模型调用、流解码、来源调查、对话投影、历史预算、滚动摘要、无状态文本服务调用和工具批执行**。它运行在 DeepTutor 私有的 Node 子进程中。Python 应用仍负责认证、持久化轮次、会话存储、能力准入、伙伴通道和未迁移的工具。浏览器协议和存储的会话格式保持兼容。

## 运行与回滚

需要 Node 22+。无生产 npm 依赖。

```sh
cd packages/chat-engine
npm ci
npm test
cd ../..
DEEPTUTOR_CHAT_ENGINE=typescript-native deeptutor start
```

| 配置 | 执行方式 |
| --- | --- |
| `python`（默认） | 参考 Python 调度器与效果；仅安装 Python 也可运行 |
| `typescript` | 第一阶段 TS 调度器，Python 负责模型/上下文/工具效果 |
| `typescript-native` | 已准入的提供商使用原生 TS 执行，其余使用兼容执行 |

选择配置并重启即可切换引擎，无需数据迁移。`DEEPTUTOR_NODE_EXECUTABLE` 用于选择 Node。构建缺失或 worker 失败会明确报错；引擎绝不会在 Python 中静默重放进行中的轮次。此源码实验尚未打包进 Python wheel。

原生准入目前覆盖标准的单密钥 OpenAI 兼容 **Chat Completions** 提供商，包括已配置的 DeepSeek 模型。Responses API、提供商特定的传输格式、托管/OAuth 认证、密钥池、Azure 以及自定义代理/CA/TLS 设置使用兼容执行，在效果执行前选定。结果会报告 `execution_runtime`，适用时附带 `compatibility_reason`。这是有意设定的兼容边界，并非完整的提供商对等。

## 模块边界

```text
CLI / WebSocket / 伙伴通道
           │
Python TurnApplicationService — 持久化日志、认证、作用域、分支选择
           │
原生种子 + 作用域应用端口
           │
TS NativeSession
   ├── core/run-loop       纯调度、预算、暂停与结算
   ├── context/            历史选择、摘要、请求投影
   ├── providers/          HTTP、SSE、用量、取消与准入限制
   └── tools/              批编排与原生实现
           │
Python 检查点/事件端口与单个遗留工具调用
```

| 模块 | 职责 |
| --- | --- |
| `src/core/` | 调度器契约与状态转换；无 I/O 或提供商依赖 |
| `src/native/` | 每轮会话与流式模型轮次生命周期 |
| `src/context/` | 完整历史分组、摘要、快照、工具/图片重放与预算投影 |
| `src/providers/` | 有界 HTTP/SSE 解析、提供商兼容与共享并发限制 |
| `src/tools/` | 准入、去重、私有字段过滤、执行与结果排序 |
| `src/transport/stdio.ts` | 多路复用私有 IPC、取消与背压 |
| `deeptutor/runtime/ts_engine/` | Worker 生命周期、预检提供商路由与无工具补全适配器 |
| `deeptutor/agents/loop/native_seed.py` | 服务端拥有的作用域、已准入工具、提示片段与初始种子 |
| `deeptutor/agents/loop/native_adapter.py` | 应用检查点/事件、能力策略与作用域遗留工具 |
| `deeptutor/agents/loop/native_tools.py`、`native_memory.py` | 服务端拥有的模型/来源/记忆授权 |
| `deeptutor/capabilities/explore_context/native.py` | 共享 TS 调查循环的来源准入与仅思考事件 |
| `deeptutor/services/llm/provider_core/typescript_provider.py` | 由原生文本传输支撑的现有服务 API；富请求预检 |
| `deeptutor/services/session/native_history.py` | 选中分支行与成功摘要持久化 |

原生工具：`workspace_list`、`workspace_read`、`workspace_search`、`read_skill`、`ask_user`、`read_source`、`reason`、`brainstorm`、`read_memory` 和 `partner_read`。工作区路径始终限制在已准入的根目录内；技能、来源 ID 和记忆根由作用域应用提供，绝不由模型提供的私有参数选择。记忆读取为实时、只读，且限于四个 L3 文档（每文件 2 MiB）。伙伴共享/关系记忆保持隔离。插件替换和自定义能力增强保留其实现。

`reason` 和 `brainstorm` 复用原生提供商，使用服务端选定的提示和参数；观察结果绝不流入主答案，子调用用量计入。附加来源调查复用 `NativeSession`，使用只读的 `read_source` 接口和五轮预算。它仅发出思考/追踪事件，绝不覆盖答案的转录。调查失败时可尝试只读、无工具的原生简报；绝不回退到 Python 模型 I/O。

通过服务 LLM 工厂的无状态文本调用在支持的配置上也使用 TS，包括建议和记忆提取模型调用。Python 功能仍拥有其工作流和写入。图片请求、工具/提供商状态重放、SDK 特定请求选项、提示缓存和模型前缀重写保留 SDK 兼容，在请求**之前**选定。原生重试由 Node 负责并遵守服务重试延迟；部分结果之后不再进行 Python 重试。

`ask_user` 规范化和草稿流式传输在 TS 中运行；应用保留等待与恢复。写入、沙箱执行、RAG/媒体处理、笔记本和日历存储、MCP、通道投递和领域特定工具策略仍为作用域应用端口。TS 也负责这些工具的批编排。

上下文投影与持久化历史分离：压缩不能抹除已完成效果的记录。失败/不完整的摘要不会推进水位线。Token 预算使用保守的**字符估算**，包含图片配额；提供商用量仍是计费权威。此估算并非精确分词器，可能比参考路径保留更少的轮次。

## 失败与资源行为

- 每个 Python 事件循环一个 worker，最多 128 个已准入运行；队列、帧、响应体、工具参数和并发模型操作均有界。
- 原生 IPC 通过私有管道传输提示、凭据和结果，绝不可记录日志。子进程接收最小环境；API 密钥按请求作用域化，不作为全局 worker 凭据继承。
- 事件在应用发布后确认，提供背压。相邻内容/思考增量在 16 ms 或 8 KiB 内合并；首个 token、追踪变更和工具/状态边界立即刷新。延迟答案也使用有界块。工具结果按完成顺序到达 UI，而模型历史保留源顺序。每个完成结果在其完成事件之前更新应用检查点，取消时保留已完成的工作。
- 截断的模型输出不能执行该批中的任何工具，即使某个参数对象是有效 JSON。暂停和重新绑定工具保留特殊顺序。
- HTTP 连接/读取不活动上限为 90 秒。部分模型输出绝不自动重试。取消会中断网络工作和待处理的主机调用。用户等待没有人为的模型响应超时。
- Worker 丢失会使活动轮次失败且不重放效果。持久化日志保留应用持久化的事件。临时模型检查点更新 Python 的轮次运行时；SQLite 持久化仍属于应用生命周期，因此这**不是**每工具崩溃持久的交易日志。中断的调用携带明确的未知结果占位符。未实现自动崩溃恢复。
- 空闲 worker 在无活动工作 30 秒后退出，释放内存。等待用户输入计为活动工作。下一轮需支付 worker 启动成本。
- 原生模型限制在 worker 内按提供商身份共享。Python 兼容流量有自己的限制器。这不是分布式速率配额。
- 检查模型工具准入和必填字段；工具实现保留参数验证。通用 JSON Schema 验证器尚未包含在本包中。

原生 API 配置使用生成的 `deeptutor/api/router_index.json` 惰性加载现有路由组；认证/依赖和 OpenAPI 仍属于原始路由。过期的源指纹回退到急切注册。路由编辑后使用项目的 Python 环境重新生成：

```sh
python scripts/build_router_index.py
```

原生路径还避免为简单聊天、来源调查、推理工具、历史摘要、标题和已准入文本服务加载 Python OpenAI SDK 和分词器。读取嵌入配置不再在后台目录扫描期间导入嵌入适配器或其 SDK。精简的 Node HTTP 传输使用有界 keep-alive 池；不添加第二个 HTTP 客户端 SDK。加载其他功能仍可能增加 Python 进程内存。

## Pi 源码审查

审查了 [Pi](https://github.com/earendil-works/pi/tree/482c1381a00d8d00bc81697e4a980c593db6803e) 的完整检出，重点检查了 agent-core、提供商/流工具、持久化压缩以及编码代理会话/压缩边界。这是源码比较，不是 Pi 运行时性能基准。未添加 Pi 依赖或复制的实现。

本轮采纳的具体规则：

1. 模型输出截断后拒绝整个工具批
   （[agent loop](https://github.com/earendil-works/pi/blob/482c1381a00d8d00bc81697e4a980c593db6803e/packages/agent/src/agent-loop.ts)）。
2. 及时发布并发工具结果，同时保留模型转录顺序
   （[tool orchestration tests](https://github.com/earendil-works/pi/blob/482c1381a00d8d00bc81697e4a980c593db6803e/packages/agent/test/agent-loop.test.ts)）。
3. 仅接受干净、非空、无工具的摘要后才推进持久化水位线
   （[durable compaction](https://github.com/earendil-works/pi/blob/482c1381a00d8d00bc81697e4a980c593db6803e/packages/durable/src/harness/compaction.ts)）。

Pi 还将模型转换与应用消息分离并惰性加载提供商；这些原则与本包的投影和适配器边界一致。其 Schema 验证、排队转向和持久化每工具任务模型是有用的下一步边界，但导入完整 harness 会重复当前持久化并引入提供商 SDK 依赖。这些需要单独的度量迁移。DeepTutor 的账户/工作区作用域和事件背压保留在此。

## 验证与度量

```sh
cd packages/chat-engine && npm test
cd ../..
DEEPTUTOR_CHAT_ENGINE=typescript pytest -q tests/agents/chat \
  tests/runtime/test_ts_engine.py tests/services/partners/test_partner_runtime.py \
  tests/capabilities/test_partner_group_capability.py \
  tests/capabilities/test_mastery_loop_pipeline.py
DEEPTUTOR_CHAT_ENGINE=python pytest -q tests/runtime/test_native_engine.py \
  tests/api/test_router_loader.py tests/app/test_turn_application_service.py \
  tests/api/test_unified_ws_turn_runtime.py tests/capabilities/test_explore_context_capability.py \
  tests/services/llm tests/services/embedding
python scripts/check_architecture.py
```

原生测试使用真实 Node 管道和本地 HTTP 模型 fixture，覆盖并发作用域隔离、取消、worker 丢失、部分流、截断、历史水位线、工作区限制、ask-user 草稿和排序。路由测试比较急切/惰性认证、依赖覆盖、路由行为、WebSocket 和 OpenAPI。

使用**相同的项目 Python 环境**、干净的测试主目录、相同的 mock 提供商延迟以及包含 Node 的进程树 RSS 来比较配置。分别报告冷启动/首次响应、热响应和空闲后内存。惰性导入和消除重复的 Python SDK/分词器生命周期解释了大部分观察到的收益；仅改变语言不会消除模型网络延迟。前一分支的通知唤醒、空闲 GC 和插件发现优化对两种配置均有益。