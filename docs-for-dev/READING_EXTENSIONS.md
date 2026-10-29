# 沉浸式阅读扩展

[手册](README.md) · [相关指南](PLUGINS.md)

沉浸式阅读通过 `deeptutor.reading_extensions` Python 入口点组发现服务端包。DeepTutor 在该组中内置朗读、学习指导、词汇、测验和显式目标翻译扩展；未安装任何扩展时，阅读器不渲染扩展工具栏。

入口点解析为具有经过校验的 `manifest` 和 `run_action(action, context)` 方法的对象或类。当前协议版本为 `1`。

```toml
[project.entry-points."deeptutor.reading_extensions"]
example = "example_reading_plugin:ExampleExtension"
```

```python
from deeptutor.reading.extensions import (
    ReadingAction,
    ReadingExtensionManifest,
    ReadingExtensionResult,
)


class ExampleExtension:
    manifest = ReadingExtensionManifest(
        id="example",
        version="1.0.0",
        name="Example",
        actions=[ReadingAction(id="explain", label="Explain")],
        result_types=["card"],
    )

    def run_action(self, action, context):
        return ReadingExtensionResult(
            type="card",
            title="Example",
            payload={"body": context.visible_text[:500]},
        )
```

## 安全边界

- 全局阅读 API 认证策略保护扩展路由。
- 服务端解析材料、定位符、已保存的源锚点和存储的单元文本；浏览器无法用任意值替换它们。
- 仅当选区在存储的单元中逐字出现时才会被转发。
- 扩展返回四种经过校验的结果类型之一：`card`、`quiz`、`feedback` 或 `browser_speech`。
- 结果序列化上限为 64 KB。测验和语音载荷还需接受额外的结构和长度校验。
- 超过协议 60,000 字符上下文上限的单元会被拒绝并返回客户端错误，而不会调用扩展。
- 操作执行超时为 30 秒，超时后返回标准的可恢复不可用响应。已在运行线程中的同步 Python 处理器无法安全终止，因此每个扩展拥有一个私有工作线程，超时后其熔断保持打开；后续调用快速失败，而不会占用或排队到进程级线程池。修复或移除卡住的扩展后，请重启 DeepTutor。
- 结果数据以 React 文本渲染。扩展无法向阅读器发送 JavaScript 或原始 HTML。
- 发现和执行失败相互隔离。损坏的可选包不会阻止文档或其他扩展打开。

协议变更必须在版本 `1` 内保持向后兼容。未来不兼容的契约必须使用新的协议版本，而不能改变现有字段的含义。

## 实现与验证

- [deeptutor/reading/extensions.py](../deeptutor/reading/extensions.py)