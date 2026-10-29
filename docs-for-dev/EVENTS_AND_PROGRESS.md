# 事件、持久化重放与索引进度

[手册](README.md) · [架构](ARCHITECTURE.md) · [索引恢复](INDEXING_RECOVERY.md)

不同事件系统的持久化保证各不相同。先从正在诊断的操作入手；进程内通知不是持久化的轮次日志。

| 流程 | 主要归属 | 检查内容 |
| --- | --- | --- |
| 模型轮次 | [应用服务](../deeptutor/app/service.py)、[轮次运行时](../deeptutor/services/session/turn_runtime.py)、[StreamBus](../deeptutor/runtime/stream_bus.py) | 提交标识、持久化序列、重放游标、终止事件 |
| 多 Worker 协调 | [协调器](../deeptutor/runtime/coordination/) | 租约持有者、隔离（fencing）、取消通知、恢复 |
| 知识摄取 | [进度跟踪器](../deeptutor/knowledge/progress_tracker.py)、[索引日志](../deeptutor/knowledge/indexing_run.py) | 任务标识、来源结果、发布状态、最后活动 |
| 知识 UI 传输 | [进度端口](../deeptutor/knowledge/progress_events.py)、[广播器](../deeptutor/api/utils/progress_broadcaster.py)、[任务流](../deeptutor/api/utils/task_log_stream.py) | 快照、WebSocket 进度、SSE 日志/终止状态 |
| 进程通知 | [EventBus](../deeptutor/events/event_bus.py) | 当前发布者/订阅者；进程生命周期 |

## 轮次生命周期

应用服务通过共享的会话存储和运行时协调准入与执行。每次执行都有一个按轮次的流总线；事件必须先进入持久化/重放路径，适配器才能将其投递给消费者。浏览器断开本身不能证明轮次失败，也不能授权第二次执行。恢复时必须区分活跃租约与丢失的 Worker，并保留终止状态。

修改流式传输时，要测试重复提交、部分输出后重连、终止事件后重放、取消以及 Worker 丢失。保留事件 ID/序列和现有投影规则，不要把到达顺序或渲染文本当作持久化的真实来源。协议类型位于 [`core/stream.py`](../deeptutor/core/stream.py)；前端使用 [`web/contracts/`](../web/contracts/) 下的契约。

单个 Worker 可以使用内存协调器。多个后端 Worker 需要 Redis 协调，通过 `system.backend_workers` 和 `integrations.turn_coordination` 配置。[设置校验器](../deeptutor/runtime/coordination/settings.py)会拒绝多 Worker 配内存协调器，也会拒绝没有 URL 的 Redis。Redis 是协调/通知层，不替代会话存储。`deeptutor doctor runtime --format json` 可输出运行时诊断信息。

## 知识库进度

`ProgressTracker` 持久化快照，并通过注入的进度端口发出事件。API 安装 WebSocket/SSE 端口；领域代码不得导入 FastAPI 来上报进度。CLI/SDK 执行仍可在没有浏览器广播器的情况下持久化进度。

知识路由暴露一个进度 WebSocket 和任务日志 SSE 流。[`useKnowledgeProgress`](../web/hooks/useKnowledgeProgress.ts) 将二者结合，跟踪预期任务并协调终止状态。重连或应用快照时，要保留工作区范围和任务标识。日志和百分比估算只是观测值，不能证明索引已发布。

[索引日志](INDEXING_RECOVERY.md)记录来源级别的成功、失败和中断的工作。完成状态必须与持久化的发布状态一致；解析器到达最后一批并不足够。重建不得用部分候选版本替换可查询版本。关于解析检查点和精确来源定位，另见[来源可视化](SOURCE_VISUALS.md)。

## 调试顺序

1. 确定账户、工作区、会话/轮次或知识库/任务 ID 以及时间窗口。
2. 先检查持久化状态，再检查协调器/Worker 和传输层。
3. 将浏览器的游标/任务标识与服务端持久化状态进行比对。
4. 用测试夹具复现连接断开、旧快照和重复帧。不要通过手动覆盖成功字段来修复进度。
5. 重新打开后验证终止状态，而不只是在原始浏览器连接仍存活时验证。

相关测试套件包括[应用测试](../tests/app/)、[协调测试](../tests/runtime/coordination/)、[知识测试](../tests/knowledge/)和[知识进度 WebSocket 测试](../tests/api/test_knowledge_progress_ws.py)。