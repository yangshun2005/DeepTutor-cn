# DeepTutor v1.0.0-beta.2 发布说明

**发布日期：** 2026.04.07

## 亮点

### 设置热重载

模型设置变更（API 密钥、模型选择、端点）现在立即生效——无需重启服务器。通过设置页面或引导流程保存后，运行时 LLM、嵌入和配置缓存会自动失效。

### MinerU 嵌套输出支持

问题提取器现在能发现嵌套 MinerU 输出目录（如 `hybrid_auto/`）中解析的 markdown，修复了 MinerU 成功解析文档但因找不到 markdown 而导致问题生成失败的情况。

### Mimic WebSocket 修复

修复了 `/mimic` WebSocket 端点因缺少 `sys` 和 `Path` 导入而导致的 `NameError` 崩溃。

### 最低要求 Python 3.11+

不再支持 Python 3.10。最低要求版本现为 **Python 3.11**。CI 矩阵、`pyproject.toml` 及所有文档已相应更新。

### CI 与维护

- 移除了 Dependabot 自动依赖更新 PR
- 精简 CI 测试矩阵至 Python 3.11 / 3.12
- 为问题提取器、mimic WebSocket 路由和设置缓存失效添加了回归测试

## 社区贡献

- **@2023Anita** — MinerU 嵌套输出修复及 Python 3.11 依赖标记（#250、#251）
- **@YizukiAme** — Mimic WebSocket 导入修复及设置缓存失效（#253、#254）

**完整变更日志**：https://github.com/HKUDS/DeepTutor/compare/v1.0.0-beta.1...v1.0.0-beta.2