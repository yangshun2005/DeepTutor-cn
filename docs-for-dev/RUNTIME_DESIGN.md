# 运行时设计：激活、Worker 与模型历史

[手册](README.md) · [架构](ARCHITECTURE.md) · [事件](EVENTS_AND_PROGRESS.md)

本页保留早期设计笔记中已实现的决策。它描述当前边界；历史内存测量值和单机测试计数不构成性能保证。

## 惰性激活

工具/能力目录使用轻量导入描述符。注册表检查不得急切导入未使用的提供者、解析器或可选模块。注册表按需加载实现；模块选择和插件审批在激活时仍然生效。兼容性导出可使用惰性包装器以保留现有导入路径。

前端同样延迟加载设置编辑器。更改加载方式时，须保留锚点/导航激活和认证单飞行为。用导入边界测试和前端路由预算验证，而不只是热进程计时。源码归属：[注册表](../deeptutor/runtime/registry/)、[模块组装](../deeptutor/modules/) 和 [设置](../web/features/settings/)。

## 短生命周期解析 Worker

重型同步提取即使在 线程中运行，也可能在长期存活的 API 进程中保留原生库内存。[隔离 Worker 边界](../deeptutor/runtime/isolated_worker.py) 通过 [`worker_process.py`](../deeptutor/runtime/worker_process.py) 启动短生命周期解释器，使用内部可导入的可调用对象和私有临时输入/结果文件。

这是受信任的内部 IPC。其序列化的 Python 对象不是面向不可信上传或网络输入的 API。客户端不能选择任意可调用对象。子进程边界在退出时释放内存，但其本身不是针对不可信代码的 OS 安全沙箱。

超时和取消必须在返回前终止并 join 进程树。并发受 `DEEPTUTOR_ISOLATED_WORKERS` 限制（默认两个）；调用点可覆盖默认超时。保持平台特定清理和结果/错误恢复由 [Worker 测试](../tests/runtime/test_isolated_worker.py) 覆盖。解析器失败必须与空成功提取保持可区分。

## 私有模型历史

显示消息和提供者就绪的模型历史用途不同。[`model_history.py`](../deeptutor/services/session/model_history.py) 保留带版本的私有 `model_turn`，包含准备好的输入、快照、模型/工具消息、重放元数据和请求指纹。公开会话响应和导出会将其脱敏。没有有效模型元数据的旧行保留其显示历史回退。

共享循环将变化的运行时事实作为命名快照追加，同时保持常驻指令稳定。替换一个快照会改变该部分；它不得将源材料重新解释为系统策略。工具可用性、工作区路径和资源权限仍来自其正常权威来源。

保留/压缩必须将模型轮次的工具调用和结果保持在一起。被中断的工具获得明确的不可用结果，而非伪造的成功响应。路由变更会移除不兼容的私有提供者重放字段，同时保留公开证据。变更必须保留存储的对话内容以及账户/工作区隔离。

## 缓存证据

请求缓存诊断比较规范请求指纹和未变前缀消息。它们不证明提供者提供了缓存令牌。实际提供者用量计数器是计费/缓存命中行为的证据；路由变更、驱逐、工具变更和压缩可能使重用失效。

从 [模型历史回归测试](../tests/services/session/test_model_history.py) 和 [请求缓存测试](../tests/services/llm/test_request_cache.py) 开始。可选的 [提示缓存集成测试](../tests/integration/test_prompt_cache.py) 使用合成输入和配置的真实提供者：

```bash
DEEPTUTOR_CACHE_E2E=1 python -m pytest tests/integration/test_prompt_cache.py -q
```

这会产生提供者用量。通过的合成缓存探测不是生产延迟或成本基准，且阶段特定流水线可能具有与共享聊天循环不同的历史契约。