# 使用你的 Agent 安装并排查 DeepTutor 问题

[手册](README.md) · [开发](DEVELOPMENT.md) · [模块](MODULES.md) · [存储](DATA_STORAGE.md)

本指南面向外部 Agent，用于为用户安装、配置和验证 DeepTutor，或排查已有安装中的问题。若问题已存在，请从[故障排查](#6-故障排查)开始，不要重复安装。CLI 概览见 [SKILL.md](../SKILL.md)。你需要终端和文件访问权限；本流程不需要 MCP 或 Agent 专用插件。

## 1. 确认目标

只询问缺失的信息，并在等待期间继续做独立准备：

- 安装目录，以及是否已有 DeepTutor 检出。
- 完整 Web 应用还是仅 CLI；按 README 安装提示默认选完整应用。
- 目标版本或分支。本指南描述当前 `dev`，包括 1.7.0 模块化布局；旧稳定版请使用对应文档。
- LLM 提供商、准确的模型 ID，以及自定义/本地端点（如有）。
- 凭据来源：已有环境变量、用户指定的私有本地文件，或提供商登录。不要在聊天中索要密钥。
- 现在是否需要知识库/RAG；这些还需要嵌入模型。搜索为可选项。

复用合适的检出/环境，保留本地修改和设置。使用用户已有的 `DEEPTUTOR_HOME`，或明确选择一个运行时主目录。它拥有 `data/`；它本身不是 `data/` 目录，也不是用于学习资料的内容工作区。

## 2. 克隆并安装

全新安装当前 `dev` 完整应用（macOS/Linux 示例）：

```bash
git clone --branch dev https://github.com/HKUDS/DeepTutor.git
cd DeepTutor
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements/full.txt
python -m pip install -e . --no-deps
cd web
npm ci --legacy-peer-deps
cd ..
```

本流程使用 Python **3.11–3.14** 和 **Node.js 22 LTS**。创建环境前先检查实际解释器；`python3` 可能版本过低。Windows 上使用受支持的 `py -3.X` 和 `.venv\Scripts\Activate.ps1`。若 shell 状态无法在工具调用之间保持，请用绝对路径调用该环境的 Python 和 `deeptutor`，并向每个进程传入环境变量。

从源码使用 CLI 时，保留上述 Python 安装并跳过 Node/npm。源码树会暴露所有已检入的模块，因此仍需其 Python 依赖。在 1.7.0 包布局中，`deeptutor` 提供最小 CLI；`deeptutor-cli` 是同一核心的兼容别名。按需添加功能模块；这不是已发布 1.6.15 包的布局。

1.7.0 包布局使用 `python -m pip install "deeptutor[full]"` 安装完整 Web 应用。裸 `deeptutor` 只含核心聊天、CLI 和 SDK；不含服务器或 Web 资源。开发构建请使用上述源码流程，或 [模块](MODULES.md) 中描述的本地 wheel。

从 1.6.15 升级时，先停止 DeepTutor，备份现有运行时主目录，并继续使用同一 `DEEPTUTOR_HOME`。安装 1.7.0 时运行 `python -m pip install --upgrade "deeptutor[full]==1.7.0"`，重启前先运行 `python -m pip check` 和 `deeptutor module list`。若已安装旧版 `deeptutor-cli` 包，请在安装新包**之前**卸载：`python -m pip uninstall deeptutor-cli`。其旧 wheel 拥有共享 Python 文件，一条命令升级可能删除新安装的核心。这不会删除运行时主目录数据。之后可在安装命令中加入 `deeptutor-cli==1.7.0`，获得新的仅元数据兼容别名。仅 CLI 迁移时，移除旧 CLI 后安装 `deeptutor-cli==1.7.0`，并添加所需功能模块。

在 1.7.0 中，升级裸 `deeptutor` 只选择核心；使用 `[full]` 保留完整应用功能集。升级期间不要重置设置或更改运行时主目录。对已有检出，更新前检查其分支、提交和本地修改；不要静默切换分支或替换环境来匹配本示例。


## 3. 无提示初始化

初始化、配置、诊断和启动使用同一运行时主目录。本示例使用检出根目录；若用户已有主目录，请改用该目录：

```bash
export DEEPTUTOR_HOME="$PWD"
deeptutor init --non-interactive
deeptutor config providers
```

`init --non-interactive` 创建缺失的默认值，不选择模型、不联系提供商，并保留已保存设置。也接受 `--cli`；两种模式都不运行向导。仅初始化不能建立模型就绪状态。

`config providers` 以 JSON 打印支持的 LLM、嵌入和搜索提供商，含默认端点和 LLM API 格式。使用其中的提供商 ID。`init`、`config apply` 和 `config show` 也接受 `--home PATH`；其他命令请一致使用 `DEEPTUTOR_HOME`。

## 4. 应用配置

写入 UTF-8 JSON 文件（如 `setup.json`），将模型占位符替换为用户的选择：

```json
{
  "llm": {
    "provider": "openai",
    "model": "YOUR_MODEL_ID",
    "api_key_env": "DEEPTUTOR_SETUP_LLM_KEY"
  },
  "system": {
    "backend_port": 8001,
    "frontend_port": 3782
  }
}
```

`api_key_env` 指定的变量必须存在于 CLI 进程环境中。从用户授权的来源读取，不要回显。JSON 包含变量名，不是字面密钥。解析后的密钥保存在 DeepTutor 的私有设置中供后续启动使用。项目根目录的 `.env` 文件不会自动加载。

```bash
deeptutor config apply setup.json --check
deeptutor config apply setup.json
deeptutor config show
```

`--check` 校验输入、提供商、指定凭据、端点和端口冲突，不写入设置、不联系提供商。两种模式都输出 JSON；无效输入以代码 2 退出。应用会为每个提供的服务选择专用的 **Agent Setup** 配置档。重复应用会更新这些相同配置档；其他配置档和省略的服务/设置会保留。每次都要提供完整配置档，包括凭据引用。端口更改在重启后生效。

| 部分 | 必填字段 | 可选字段 |
| :--- | :--- | :--- |
| `llm` | `provider`、`model` | `base_url`、`api_key_env`、`api_version`、`api_format` |
| `embedding` | `provider`、`model` | `base_url`、`api_key_env`、`api_version`、正整数 `dimension` |
| `search` | `provider` | `base_url`、`api_key_env` |
| `system` | 至少一个端口 | `backend_port`、`frontend_port`（1–65535 的整数，且互不相同） |

远程密钥认证的 LLM/嵌入提供商需要 `api_key_env`；本地、OAuth 和 `custom` 端点可省略。搜索要求来自 `config providers`。无默认端点的提供商需要 `base_url`。LLM API 格式为 `auto`、`openai_chat`、`openai_responses` 和 `anthropic`，受提供商支持列表约束。未知字段、字面 `api_key` 字段和 URL 中的凭据会被拒绝。高级设置和其他服务类型仍可在 Web 设置中使用。

知识库/RAG 还需配置可访问的嵌入模型。例如：

```json
{
  "embedding": {
    "provider": "openai",
    "model": "text-embedding-3-small",
    "api_key_env": "DEEPTUTOR_SETUP_EMBEDDING_KEY",
    "dimension": 1536
  },
  "search": { "provider": "duckduckgo" }
}
```

嵌入 `base_url` 是完整端点，如 `https://api.openai.com/v1/embeddings`；目录会像 Web UI 一样规范化它。更改现有知识库的嵌入模型/维度需要规划重建。不需要搜索时省略，或选 `none` 禁用。DuckDuckGo 无需密钥；可用性取决于用户网络。

本地 Ollama 使用 `{"llm":{"provider":"ollama","model":"YOUR_LOCAL_MODEL"}}`，并确认 Ollama 正在运行且模型可用。

OAuth 请从 `config providers` 应用提供商 ID/模型，然后在同一运行时主目录使用其现有登录命令。用户完成所有登录交互：

```bash
deeptutor provider login openai-codex
# 其他支持的流程：github-copilot、codebuddy
```

OpenAI Codex 使用 DeepTutor 自己的 OAuth 凭据；外部 Agent 的登录不会自动复用。登录不会选择模型。

## 5. 验证并启动

```bash
python -m pip check
deeptutor module list
deeptutor doctor --format json
deeptutor doctor --online --format json
```

`pip check` 检查依赖一致性。检查 `module list` 中用户所需的模块：已安装、已启用和就绪是不同状态。Web 应用需要 server 和 web；RAG 需要 knowledge 和 parsing；可选能力需要其所属模块。仅核心会故意省略这些模块。源码检出使用 `module enable/disable`；wheel 安装还支持 `module install/uninstall`。更改前停止 DeepTutor，未发布模块使用 `--wheelhouse /path/to/wheels`。见[模块](MODULES.md)。

`doctor` 检查本地就绪状态；`doctor --online` 发起一次小型真实模型请求，可能产生提供商用量。检查退出状态和必需项失败。修复可操作的失败并重跑受影响的检查。仅保存设置不能证明就绪。LLM 探测不验证可选嵌入：需要 RAG 时，还要在设置中测试其连接，并摄取/搜索一份用户批准的小文档。

完整应用：

```bash
deeptutor start --detach --no-browser
```

读取实际启动器 URL，验证后端 `/health/ready`，并确认前端有响应。默认后端 `http://127.0.0.1:8001`，前端 `http://127.0.0.1:3782`；若不同请使用报告的端口。失败时检查启动器日志。通过 `config apply` 选择空闲端口，不要停止无关进程。仅 CLI 时跳过 Web 启动，验证轮次执行：

```bash
deeptutor run chat "Reply with a short greeting." --format json
```

这也会使用已配置的模型。检查响应成功和轮次完成，而不仅是零退出码或首个流式事件。

最后报告检出/环境、运行时主目录、提供商/模型（不含凭据）、通过的检查、Web URL 或 CLI 调用方式，以及剩余用户操作。分离模式应用需附上同一主目录的 `deeptutor stop`。若凭据或网络访问阻止验证，请说明确切未完成的步骤，不要宣称安装完成。

## 6. 故障排查

按以下方法排查用户症状。根据证据选择检查项；无需为每个问题运行所有命令。

1. **确认运行实例并复现症状。** 确定安装方式、已安装版本或源码提交、活动 Python 环境、运行时主目录和实际服务 URL。Docker 需区分容器路径/端口与主机路径/端口。询问失败操作、预期结果、大致时间和相关近期更改。确认你的 CLI 检查针对与用户相同的实例、账户和工作区；另一个健康的安装对故障帮助不大。

2. **收集最小有用证据。** 用 `config show` 和 `doctor` 检查配置/就绪状态，用 `doctor --online` 发起真实 LLM 请求，或用 `doctor runtime` 检查轮次协调、会话存储和迁移预检。两种 doctor 模式都可加 `--format json`。检查单项：顶层 `ok: true` 可能与 RAG 等可选项失败并存。LLM 探测通过不能证明嵌入、搜索、工具或浏览器连接正常。

3. **沿失败操作的边界追踪。** 从可见症状出发，确定操作进行到哪一步：浏览器 → API → 轮次/能力 → 提供商、工具或存储。UI 问题检查浏览器控制台和网络错误，并按时间和 `request_id`、`session_id` 或 `turn_id`（如有）关联服务器日志。分离启动时检查 `<runtime-home>/data/user/runtime/launcher.log`；应用日志通常在 `data/user/logs/deeptutor.jsonl`，受日志设置影响。容器部署可能还需容器输出。只读相关窄段，分享前移除凭据和私有内容。

4. **一次测试一个假设。** 将失败操作缩小为小型复现，对比正常路径与失败路径。例如 CLI 轮次成功而 Web 轮次失败，若两者设置和上下文等价，则指向浏览器/API 路径。用 `--help`、`plugin info` 和检出的代码继续调查。根 `AGENTS.md` 映射架构；有用起点包括 `deeptutor_cli/`、`deeptutor/api/`、`deeptutor/runtime/`、`deeptutor/capabilities/`、`deeptutor/services/` 和 `web/`。得出结论前将源码与已安装版本匹配。

5. **做聚焦修复并验证原始操作。** 保留现有数据和配置；避免将全面重装、重置或知识库重建作为探索步骤。更改后重跑用户的失败操作，而不只是通用健康检查。将观察与假设分开报告。若未解决，提供简明交接：环境/版本、复现步骤、预期与实际行为、相关脱敏错误、已尝试的检查，以及下一步有用调查。

## 实现与验证

- [deeptutor_cli/setup_config.py](../deeptutor_cli/setup_config.py)
- [deeptutor_cli/doctor.py](../deeptutor_cli/doctor.py)
- [tests/cli/test_agent_setup.py](../tests/cli/test_agent_setup.py)