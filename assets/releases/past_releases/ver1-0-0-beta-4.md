# DeepTutor v1.0.0-beta.4 发布说明

**发布日期：** 2026.04.10

## 亮点

### 嵌入进度跟踪与速率限制重试
知识库初始化期间新增实时嵌入进度报告——UI 现在会显示 `batch N/M complete`。HTTP 429（Too Many Requests）响应会自动以指数退避重试，可配置的 `batch_delay` 参数让免费用户能限制请求频率以保持在速率限制内。进度回调在 `finally` 块中正确清理，防止泄漏到后续搜索调用。

### 跨平台启动引导依赖管理
入门启动引导现在会在缺失时自动安装引导依赖（如 PyYAML），并支持在 macOS（Homebrew）、Linux（apt/dnf/yum）和 Windows（winget/Chocolatey）上安装系统依赖，用于 Math Animator 的 LaTeX、FFmpeg、Cairo 和 CMake 等前置条件。`typer[all]` 依赖也已简化为 `typer`，避免拉取不必要的附加项。

### 大小写不敏感的 MIME 验证
修复了一个平台相关 bug：大写扩展名的文件（如 `report.PDF`、`data.JSON`）在 Linux 上会绕过 MIME 类型验证。`mimetypes.guess_type()` 现在接收小写文件名，与扩展名白名单检查保持一致。

## 社区贡献

- **@oxkage** — 嵌入进度跟踪与 HTTP 429 速率限制重试（#268）
- **@kuishou68** — 大小写不敏感 MIME 类型验证修复（#272，关闭 #271）

**完整变更日志**：https://github.com/HKUDS/DeepTutor/compare/v1.0.0-beta.3...v1.0.0-beta.4