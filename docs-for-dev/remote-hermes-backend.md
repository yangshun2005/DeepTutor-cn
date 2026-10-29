# Remote Hermes Agent 后端

[手册](README.md) · [相关指南](ARCHITECTURE.md)

DeepTutor Connected Agents 无需在 DeepTutor 容器内安装 Hermes CLI，即可调用 Hermes Agent 网关。该后端类型为 `hermes_remote`，显示名称为 **Hermes Agent (remote)**。

## 配置

各后端设置位于常规 `subagent.json` 设置文档的 `backends.hermes_remote` 下：

- `enabled`：聊天是否可委派给此后端。
- `base_url`：网关根地址，例如 `http://hermes-uni:8642`。
- `api_key_env`：存放网关 bearer 的环境变量名，默认 `DEEPTUTOR_HERMES_REMOTE_API_KEY`。
- `profile`：部署/配置标签，仅作说明；它不是凭据，也不用于构造 URL。
- `model`、`effort`、`system_prompt`、`auto_approve`：与本地后端共享的运行控制项。
- `idle_timeout_seconds`：SSE 帧之间的最大空闲间隔（默认 `600`）。网关的 keepalive 会重置此看门狗。

API key 仅通过环境变量提供。它从不作为内联设置字段被接受、持久化，也不会由 `GET /api/subagents/settings` 返回。
网关 URL 必须为 HTTP(S)，且不含凭据、查询文本或片段。可选的 `/p/<profile>` 前缀支持 Hermes 多配置路由。不跟随重定向。

## HTTP 契约

适配器使用带认证的 bearer 请求（`Authorization: Bearer ...`），并显式设置连接/读取/写入/连接池超时。端点和机器可读的功能契约见
[Hermes API Server 指南](https://hermes-agent.nousresearch.com/docs/user-guide/features/api-server/)。

1. `GET /v1/capabilities` 检测可用性，并验证配置的网关是否声明支持运行提交、事件流、停止、审批响应和会话资源。检测会报告不同的 `not_configured`、`key_missing`、`unreachable`、`unauthorized` 和 `incompatible` 详情。
2. `POST /v1/runs` 启动运行。请求使用 `input` 传用户提示，`instructions` 传可选的首次会话系统指令，配置了 `model` 时传 `model`，配置了 `model_options.reasoning_effort` 时传该项，恢复会话时传 `session_id`。服务器返回 HTTP 202 及 `{ "run_id": "...", "status": "started" }`。新运行使用其 run id 作为会话 id；恢复的运行保留提供的会话 id。
   恢复运行前，适配器会获取 `GET /api/sessions/{session_id}/messages`。它只保留非空的 user/assistant 行，排除 tool 行，并最多将最后 40 行作为 `conversation_history` 发送。404 表示会话已不存在：丢弃旧会话 id 和请求头，运行在无历史的情况下重新开始，并再次应用 `system_prompt`。
3. `GET /v1/runs/{run_id}/events` 是 SSE 流。帧在 `data:` 行中包含 JSON。适配器消费以下事件对象：
   `message.delta`（`delta`）、`tool.started`（`tool`、`preview`）、
   `tool.completed`（`tool`、`error`）、`reasoning.available`（`text`）、
   `approval.request`、`run.completed`（`output`）、`run.failed`（`error`）和
   `run.cancelled`。工具生命周期事件转为 tool/tool-result 行；
   回答增量在合并 id `hermes_remote:final` 下转为累积文本；其他生命周期事件转为 log/error 行。网关通过同一流发出工具/推理、文本/完成和审批事件。
4. `POST /v1/runs/{run_id}/approval` 处理待审批请求。自动批准时请求体为 `{ "choice": "once" }`，`auto_approve` 为 false 时为 `{ "choice": "deny" }`。审批响应失败会暴露错误并停止运行，而不是无限等待。
5. `POST /v1/runs/{run_id}/stop` 中断运行。取消时以受保护、有界的清理窗口发送此请求，然后重新抛出取消；SSE 空闲超时会发送此请求并返回错误。

会话连续性同时使用网关文档中的 `X-Hermes-Session-Id` 请求头和 `session_id` 请求字段。适配器还会发送兼容的 `X-Hermes-Session` 请求头。自定义 `system_prompt` 在新会话或历史查询报告会话已不存在（404）时发送。每个请求都携带固定的 `CONSULT_ORIGIN_INSTRUCTION` 递归防护，标识 DeepTutor Connected Agents，并告知网关 agent 直接回答而非委派回 DeepTutor。

`/v1/runs` API 没有图片上传字段。因此附件及其 DeepTutor 本地路径不会发送到远程网关。

## 隔离与安全

此后端将 Hermes 网关置于 DeepTutor 进程/容器之外：工具在网关主机上运行，而非 DeepTutor 内。将每个 DeepTutor 聊天限定在其返回的会话 id 上；不要跨用户或聊天共享会话 id。将 `API_SERVER_KEY`（网关的服务端密钥）和 `DEEPTUTOR_HERMES_REMOTE_API_KEY`（DeepTutor 侧环境变量名的值）排除在设置文件、日志、事件文本和前端载荷之外。

除适配器暴露的结构化 SSE 生命周期外，流式传输保持静默。它不轮询或重试 `POST /v1/runs`；调用方取消或空闲看门狗始终尝试有界停止。

## 实现与验证

- [deeptutor/services/subagent/hermes_remote.py](../deeptutor/services/subagent/hermes_remote.py)
- [deeptutor/services/subagent/hermes_remote_client.py](../deeptutor/services/subagent/hermes_remote_client.py)
- [deeptutor/services/subagent/hermes_remote_events.py](../deeptutor/services/subagent/hermes_remote_events.py)
- [tests/services/test_hermes_remote_backend.py](../tests/services/test_hermes_remote_backend.py)