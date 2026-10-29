# 工具与能力

[手册](README.md) · [架构](ARCHITECTURE.md) · [模块](MODULES.md) · [插件](PLUGINS.md)

## 两个契约

[`BaseTool`、`ToolDefinition` 和 `ToolResult`](../deeptutor/core/tool_protocol.py)
描述由模型选择的一次操作。工具校验参数、执行操作，并返回内容及结构化元数据/错误
状态。可用性不等于访问所有资源的权限。

[`TurnCapability` 和 `CapabilityManifest`](../deeptutor/core/capability_protocol.py)
描述拥有一个轮次的流水线。能力消费 `UnifiedContext` 并发出流事件。能力统一通过
[`emit_capability_result()`](../deeptutor/agents/_shared/capability_result.py) 输出
响应载荷和 `cost_summary`；不要另造完成信封。

内置目录由
[`builtin_capabilities.py`](../deeptutor/runtime/bootstrap/builtin_capabilities.py)
从 [`modules/manifests/`](../deeptutor/modules/manifests/) 派生。
用 `deeptutor module list`、`deeptutor plugin list` 和 `deeptutor plugin info`
查看当前安装，不要假设每个源码功能在仅核心 wheel 环境中都可用。

## 挂载规则

八个用户可切换工具是 `brainstorm`、`web_search`、`paper_search`、
`zotero_search`、`reason`、`geogebra_analysis`、`imagegen` 和 `videogen`。
[`tools/builtin/__init__.py`](../deeptutor/tools/builtin/__init__.py) 中的
`USER_TOGGLEABLE_TOOL_NAMES` 是权威列表；CLI `--tool` 从中选择。

`CONFIGURABLE_BUILTIN_TOOL_NAMES` 声明上下文门控工具，包括研究、检索、记忆、
笔记本、执行和延迟工具发现。
[`tool_composition.py`](../deeptutor/agents/_shared/tool_composition.py) 负责
挂载标志和始终可用的工作区工具。能力拥有的 solve、mastery 及其他专用工具遵循
各自的激活契约。Partner 允许/拒绝面可进一步收窄访问。

账户授权、工作区资源策略、模块选择和能力激活必须在发现和分发时都保持生效。
UI 或清单中的工具名不得绕过这些检查。MCP/CLI 延迟发现必须使用现有加载器，
不得安装并行注册表。

## 新增或修改内置工具

1. 在其所属模块中实现类型化操作或能力。可选库不要进入仅元数据的导入。
2. 更新工具规格或能力声明、别名及所属模块清单。延迟工具描述符位于
   [`builtin_specs.py`](../deeptutor/tools/builtin_specs.py)。
3. 将挂载逻辑放入现有组合/激活所有者。明确决定工具是用户可切换、上下文门控
   还是能力拥有。
4. 按需更新请求校验和翻译后的提示词。能力提示词位于
   [`capabilities/prompts/`](../deeptutor/capabilities/prompts/)；
   部分 agent 实现在各自目录中拥有额外提示词。
5. 测试元数据/实现一致性、缺失模块行为、授权资源范围、失败与取消，以及公共
   结果信封。按[测试](TESTING.md)执行 Schema 重新生成和更广泛检查。

对于独立分发的包，使用[插件契约](PLUGINS.md)和
[Learning Echo 示例](examples/learning_echo/README.md)，不要修改内置注册。
阅读工具栏操作有单独的[阅读扩展协议](READING_EXTENSIONS.md)。

## 诊断意外的工具调用

将活动模块状态、能力、选定资源、用户工具选择和工作区策略与模型请求实际接纳的
Schema 对比。然后检查分发和返回的证据。选定的 KB 不能证明检索已运行；检索到的
URL 不能证明像素已到达模型。参见[源视觉](SOURCE_VISUALS.md)和
[答案质量诊断](REASONING_SAFETY_CHECKLIST.md)。