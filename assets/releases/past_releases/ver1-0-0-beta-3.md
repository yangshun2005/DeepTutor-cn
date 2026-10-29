# DeepTutor v1.0.0-beta.3 发布说明

**发布日期：** 2026.04.08

## 亮点

### 移除 LiteLLM 依赖
在服务和 TutorBot 两层中，用原生 `openai` 和 `anthropic` SDK 替换了 `litellm` 抽象层。新增 `OpenAICompatProvider`（覆盖 OpenAI、DeepSeek、Mistral、StepFun、XiaoMi-MiMo、Qianfan、oVMS 等）和专用 `AnthropicProvider`。设置界面新增提供商下拉菜单，自动填充 base URL。工具调用格式出错时自动回退到流式模式（修复 #265）。

### Windows Math Animator 兼容性

修复 Windows 上 `SelectorEventLoop` 不兼容问题：用 `subprocess.Popen` + 读取线程 + `asyncio.Queue` 替换 `asyncio.create_subprocess_exec`，保留实时逐行进度输出。同时在 Windows 上应用 `ProactorEventLoop` 策略以支持子进程。

### LLM 输出的健壮 JSON 解析

七个 agent 模块（planner、idea、design、note、reporting、citation、data structures）改用 `parse_json_response()` 替代原始 `json.loads()`，正确处理被 markdown 代码围栏包裹的 LLM 响应。为回退参数引入 `_UNSET` 哨兵值，调用方可显式指定 `None` 作为失败值。

### 引导学习修复

- 修复 KaTeX 数学渲染：配置 `$...$` 和 `$$...$$` 分隔符，移除失效的 SRI 完整性哈希，为裸 LaTeX 文本节点添加父窗口回退渲染。
- 修复后端轮询（`fetchPageStatuses`）覆盖用户标签页导航的问题：仅在用户尚未导航时才接受 `current_index`。
- 引导 agent 的 `max_tokens` 从 8192 提升至 16384，防止 HTML 截断。

### 完整国际化

完成 Web UI 的 i18n 覆盖——workspace、utility、sidebar 和组件页面中所有硬编码字符串均已改为翻译键，并提供完整的英文和中文语言文件。

## 社区贡献

- **@kevinmw** — Windows Math Animator 渲染器修复、引导学习 KaTeX 渲染与轮询修复（#256、#266）
- **@LocNguyenSGU** — GitHub Copilot 提供商登录文档（#262）
- **@kagura-agent** — 用于 LLM 输出的 `parse_json_response`，处理 markdown 围栏

**完整变更日志**：https://github.com/HKUDS/DeepTutor/compare/v1.0.0-beta.2...v1.0.0-beta.3