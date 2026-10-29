# 测试与验证

[手册](README.md) · [开发](DEVELOPMENT.md) · [模块](MODULES.md#validation)

使用仓库配置的工具和 [Agent 设置](AGENT_SETUP.md) 中的环境。开发期间运行针对性检查，发布前运行受影响项目的门禁检查。仅文档变更无需付费模型调用或完整渲染负载。

## 按变更选择检查

| 变更 | 最低有效证据 |
| --- | --- |
| 文档与 CLI 示例 | 链接/锚点、CLI 文档契约、命令帮助、无过期路径 |
| Python 领域逻辑 | 针对性的 pytest 复现；Ruff；相关导入边界 |
| API 或轮次事件 | 请求/授权测试、生成的契约检查、持久化/重放测试、消费者测试 |
| 前端行为 | 类型检查、针对性的 Node/Vitest 测试、依赖检查；交互/布局需浏览器验证 |
| 模块归属/依赖 | 生成的依赖需求、模块隔离、wheel 生命周期冒烟测试 |
| 插件 worker 或打包页面 | Manifest、运行时/生命周期与 API 测试、参考示例 wheel |
| 解析/子进程代码 | 真实子进程的成功/错误/超时/取消；相关平台检查 |
| 提供商/模型集成 | Mock 契约测试，必要时加显式配置的真实提供商探测 |

## Python

```bash
python -m pytest tests/cli/test_docs_contract.py tests/scripts/test_docker_compose.py
python -m pytest tests/app tests/runtime/coordination
python -m pytest tests/plugins tests/api/test_plugins_router.py
python -m ruff check .
python -m ruff format --check .
lint-imports
python scripts/check_architecture.py
python scripts/sync_module_requirements.py --check
python scripts/check_repo_hygiene.py
```

这些是独立的入口；应选择相关套件，而非假设第一条命令覆盖所有变更。完整的 Python 套件是 `python -m pytest tests deeptutor/learning/tests`。部分集成用例需要 Redis、可选引擎或平台可执行文件等 fixture。阅读跳过条件并准确报告跳过的检查。

`check_repo_hygiene.py` 检查已跟踪路径。更严格的 `check_workspace_hygiene.py` 还要求检出完全干净，因此应在提交后运行，而非将有意编辑误判为测试失败。

## Web 与生成的契约

在 `web/` 下：

```bash
npm run typecheck
npm run architecture:check
npm run test:node
npm run test:unit
npm run lint
npm run i18n:check
npm run contracts:check
```

`npm run check:fast` 运行上述确定性检查；`npm run check` 额外包含生产构建和路由预算。修改后端 Schema 后使用 `npm run contracts:generate`，然后检查并提交生成的差异。生成器拥有 [`web/contracts/`](../web/contracts/) 并调用 [Python 导出脚本](../scripts/export_frontend_contracts.py)。

渲染组件测试使用 Vitest 和 Testing Library。Node 测试覆盖纯契约和状态逻辑。Playwright `npm run audit` 需要 `WEB_BASE_URL` 上有运行中的前端；关键项目和 multi-worker 项目在 [`playwright.config.ts`](../web/playwright.config.ts) 中有各自的 fixture 要求。不要对学习者唯一的数据副本运行浏览器变更操作。

## 模块、引擎与可选集成

按照 [模块 → 验证](MODULES.md#validation) 进行一次性 wheel 安装、卸载/重装和保留数据检查。不要用开发解释器测试包移除。

实验性 TypeScript 引擎在 [`packages/chat-engine/`](../packages/chat-engine/) 下有独立的 `npm test`，外加 Python 桥接和应用兼容性测试。标准矩阵见 [`tests.yml`](../.github/workflows/tests.yml) 和 [`modules.yml`](../.github/workflows/modules.yml)。

`deeptutor doctor --online` 执行真实模型请求。提示缓存、视频渲染和多 worker 验收测试可能需要显式启用的变量。说明实际环境和所执行的验证；历史测试计数和 mock 提供商不能证明当前生产行为。

## 审查文档变更

从每个文件出发检查相对链接，而非从仓库根目录。移动指南时搜索 README 翻译、CLI 帮助、UI 设置提示和测试 fixture。源链接避免使用易变的行号。使用已安装 CLI 的 `--help` 和权威 manifest/设置来验证命令和默认值。将本地构建产物和密钥排除在提交之外。