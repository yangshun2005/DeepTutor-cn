# Learning Echo 插件

[手册](../../README.md) · [插件契约](../../PLUGINS.md) · [测试](../../TESTING.md)

本包是托管插件运行时的参考实现。它声明了一个 Tool、一个 Capability、一个 HTTP 路由和一个静态前端页面。页面仅请求 `sandboxed-iframe` UI 权限；本包无 Python 依赖。

## 包结构

- [清单](learning_echo/deeptutor.plugin.json)：插件标识、权限及四项扩展声明。
- [Worker](learning_echo/worker.py)：描述、工具执行、能力流式传输及 JSON HTTP 响应。
- [页面清单](learning_echo/frontend/page.json)：显式静态资源闭包；[index.html](learning_echo/frontend/index.html) 和 [app.js](learning_echo/frontend/app.js) 实现沙箱页面。
- [构建元数据](pyproject.toml)：打包 worker、清单和资源。

## 构建与检查

在仓库根目录构建本地 wheel：

```bash
python -m pip wheel --no-deps -w /tmp/deeptutor-plugins docs-for-dev/examples/learning_echo
```

使用一次性的 `DEEPTUTOR_HOME`，然后用生成的 wheel 路径运行 `deeptutor plugin install`，并通过以下命令检查：

```bash
deeptutor plugin show org.deeptutor.learning_echo
deeptutor plugin state
```

在调用 `deeptutor plugin approve org.deeptutor.learning_echo` 前，先审查清单和构建产物；若插件已禁用，需显式启用。已安装与已批准是两种不同状态。安装步骤在导入插件代码前读取 `deeptutor.plugin.json`，并创建插件私有虚拟环境。

批准并启用后，页面可通过 `/api/plugins/org.deeptutor.learning_echo/pages/learning_echo_page/` 访问。其入口和资源闭包在 `learning_echo/frontend/page.json` 中声明。

启用主机认证时，打开页面或其 `public` HTTP 路由仍需主机会话。页面的 CSP 阻止任意网络访问；它不是注入主机前端代码的示例。

## 验证与适配

在仓库根目录运行：

```bash
python -m pytest tests/plugins/test_runtime.py tests/api/test_plugins_router.py
```

这些测试以本包作为 worker/页面 fixture。移动或修改示例时请保留测试。新建插件时，选择不同的包/清单标识，声明实际的权限和依赖闭包，并测试失败、取消、禁用状态及升级后重新批准。信任限制和回滚行为见[插件指南](../../PLUGINS.md)。