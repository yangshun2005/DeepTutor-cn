# 第三方插件开发

[手册](README.md) · [工具与能力](TOOLS_AND_CAPABILITIES.md) · [阅读扩展](READING_EXTENSIONS.md)

官方[模块 wheel 包](MODULES.md)与 DeepTutor 共用解释器和版本。第三方插件有独立的清单、审批和生命周期约定。请从可运行的[学习回声示例](examples/learning_echo/README.md)入手，该示例声明了工具、能力、HTTP 路由和静态页面，且不依赖外部 Python 包。

## 清单与注册

包内附带静态文件 `deeptutor.plugin.json`，使用 `deeptutor.plugin/v1` Schema。字段定义见 [`manifest.py`](../deeptutor/plugins/manifest.py) 和[示例清单](examples/learning_echo/learning_echo/deeptutor.plugin.json)。清单声明身份/版本、DeepTutor/API 兼容性、权限披露、依赖闭包和类型化扩展。未知字段和非法路径会被拒绝。

| 扩展 | 运行时约定 |
| --- | --- |
| `tool`、`capability` | 宿主入口点协议，或经批准的可管理 JSON worker |
| `loop_capability` | 现有的共享循环扩展协议 |
| `reading_extension` | 服务端校验的 Reading 上下文和有界结果类型 |
| `visualizer` | 类型化的打包可视化器声明 |
| `http_route` | 宿主插件命名空间下的可管理 worker 端点 |
| `frontend_page` | 声明的静态资源，以严格沙箱 CSP 提供 |
| `persistence_schema`、`app_connector` | 经批准的声明/内省；不提供通用迁移或连接器执行 API |

内置 ID 优先。每个扩展保留各自的类型化协议。现有 Python 入口点组在存在根清单审批时遵循该审批；新包不应依赖无清单的旧路径。发现过程只读取元数据，不导入第三方代码。[注册表](../deeptutor/plugins/registry.py)区分可用、需审批、已启用、已禁用、不兼容、损坏和已弃用状态。

## 安装与审批

生命周期命令见 `deeptutor plugin --help`。可管理安装接受本地 wheel，或从内置的 [`catalog.json`](../deeptutor/plugins/catalog.json) 解析经审核的条目。目录浏览/版本解析离线进行；显式安装会下载固定版本的构建产物。不提供任意 URL 安装器或后台自动升级。

目录中的构建产物绑定 HTTPS wheel URL、SHA-256、大小和平台元数据。下载在交给 pip 前会拒绝重定向以及大小/摘要不匹配。本地生命周期在导入代码前校验 wheel 中的清单，创建版本私有环境，并安装仅含 wheel 的显式依赖闭包。它不会改动宿主解释器。

审批绑定规范化清单和构建产物身份。审批前请检查当前快照。安装、升级、同版本重装和回滚都会清除此前的审批。审批不会自动启用本地已禁用的插件。安装失败会保留此前可用状态；保留的版本支持回滚。外部宿主安装的插件升级需重启才能替换已导入的 Python 模块。

所有者是 [`lifecycle.py`](../deeptutor/plugins/lifecycle.py)，状态位于运行时 `data/user/settings/plugins.json`。API 生命周期变更需要管理员授权。[插件测试](../tests/plugins/)覆盖清单、构建产物、事务、恢复和审批边界。

## Worker 协议与取消

[`runtime.py`](../deeptutor/plugins/runtime.py) 使用插件环境的 Python 启动 worker 模块。Worker 声明不能超出已批准的清单。旧版 worker 通过 stdin/stdout 交换一次 JSON 请求和响应。

能力可在描述中声明 `"transport": "ndjson-v1"`，随后输出 JSON 行：`{"kind":"event","event":{...}}`，最后以 `{"kind":"complete"}` 结束。事件进入常规的持久化轮次流；worker 不拥有并行的公开流。完成前遇到 EOF 或帧格式错误会使运行失败。请求、帧、输出、stderr 和执行均有上限；队列施加背压。取消会关闭进程组并等待清理。示例同时支持流式和旧版模式。

## HTTP 与静态页面

可管理 HTTP 路由使用 `/api/plugins/{plugin-id}{path}`，不能替换内置路由。宿主解析已批准、已启用、兼容的可管理状态，并在启动 worker 前完成认证。启用宿主认证时，即使插件路由声明为 `public` 也需要宿主会话：它不能绕过宿主账户边界。`admin` 策略额外增加管理员检查。不可用的插件路由返回 404。

Worker 接收声明的 method/path、规范化查询值和有界 JSON 请求体，不接收宿主 Cookie、Bearer 令牌或认证头。响应具有经过校验的 JSON 状态/响应体和受限的 `Cache-Control`；重定向和认证响应由宿主负责。参见 [API 测试](../tests/api/test_plugins_router.py)。

静态页面声明 `deeptutor.plugin-frontend-page/v1` Schema、一个入口和显式资源闭包。仅提供经批准的常规包文件；符号链接和未声明资源会被拒绝。规范 URL 为 `/api/plugins/{plugin-id}/pages/{extension-id}/`。使用相对资源时请保留末尾斜杠。

页面在带严格 CSP 的沙箱浏览器上下文中运行，包括 `connect-src 'none'`；它们不会向宿主 bundle 注入 React/Next 代码，也不会任意请求宿主 API。参见 [`frontend.py`](../deeptutor/plugins/frontend.py)、[前端测试](../tests/plugins/test_frontend.py) 和[示例页面清单](examples/learning_echo/learning_echo/frontend/page.json)。

## 信任边界与验证

权限声明和审批不是操作系统沙箱。宿主入口点扩展在宿主解释器中执行；可管理 worker 隔离 Python 依赖，但仍保留宿主操作系统访问权限。请审查实际包和依赖。打包的浏览器页面有自己的浏览器沙箱。不支持直接复用 Chrome 扩展。

验证清单、构建 wheel、演练审批/启用、worker 和页面，然后禁用/撤销并取消一次运行。使用一次性运行时主目录。参见[测试](TESTING.md)和单独的[Reading 扩展上下文/结果规则](READING_EXTENSIONS.md)。