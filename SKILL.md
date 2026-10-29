---
name: deeptutor-cli
description: 通过 CLI 配置、管理和使用 DeepTutor，涵盖能力、知识库、伙伴、记忆、会话、笔记本、提供商、技能以及服务器或 Web 应用。
---

# DeepTutor CLI 技能

> 教会你的 AI 代理完全通过命令行配置、管理和使用 DeepTutor——一个智能学习平台。

## 何时使用

当用户想要以下操作时使用本技能：
- 安装或配置 DeepTutor
- 诊断现有 DeepTutor 安装中的问题
- 与 DeepTutor 对话或运行某项能力（深度求解、题目生成、深度研究、可视化、数学动画、掌握路径）
- 创建、管理或搜索知识库
- 创建、管理或运行伙伴（IM 连接的伴侣）
- 从 Hub（EduHub 或 ClawHub）搜索、安装或管理技能
- 检查或维护交互式书籍
- 查看或管理学习记忆、会话或笔记本
- 启动 DeepTutor API 服务器或完整 Web 应用

## 前置条件

- Python 3.11–3.14
- 对于 1.7.0 模块化版本，`pip install "deeptutor[full]"` 安装打包的 Web 应用；仅安装 `deeptutor`（或 `deeptutor-cli` 兼容别名）只安装核心 CLI/SDK。开发时使用源码或本地 wheel。源码安装需先执行 `pip install -r requirements/full.txt`，再执行 `pip install -e . --no-deps`；Web UI 还需在 `web/` 下安装 Node.js 22 LTS 并运行 `npm ci --legacy-peer-deps`。从 1.6.15 升级且不改变运行时主目录的方法见安装指南。
- 首次交互式安装运行 `deeptutor init`。它会引导完成向导（端口 → LLM → 嵌入 → 搜索 → 审查），并将与 Web 设置页面相同的配置写入 `data/user/settings`。加 `--cli` 可跳过端口步骤（仅 CLI 使用），加 `--home <path>` 可指定工作区。

## 代理安装

先阅读[开发者与代理手册](docs-for-dev/README.md)，找到相关的安装、架构、开发、测试或功能指南。
当用户要求你安装、配置或排查 DeepTutor 时，遵循
[docs-for-dev/AGENT_SETUP.md](docs-for-dev/AGENT_SETUP.md)。该文档涵盖克隆、
环境准备、凭据来源、可选 RAG/搜索、启动和验证，以及针对现有安装的
[排查方法](docs-for-dev/AGENT_SETUP.md#6-troubleshooting)。修改设置前先阅读它；复用现有检出，
保留用户文件。所有命令保持 `DEEPTUTOR_HOME` 一致。

```bash
deeptutor init --non-interactive          # 创建缺失的默认值；不选择模型
deeptutor config providers               # 以 JSON 发现支持的提供商
deeptutor config apply setup.json --check # 仅校验，不写设置/不联网
deeptutor config apply setup.json         # 从 JSON 应用完整安装配置
deeptutor config show                    # 查看配置（凭据已脱敏）
deeptutor doctor --format json           # 检查本地就绪状态
deeptutor doctor --online --format json  # 小型实时 LLM 请求（消耗提供商额度）
```

使用指南中的 setup JSON schema，其中 `api_key_env` 引用 CLI 进程环境中的凭据。
切勿打印密钥或在 setup JSON 中放置明文密钥。缺少凭据/模型选择时，询问用户
来源/选择；继续独立准备工作。使用旧版本时，用已安装 CLI 的 `--help` 确认命令。
保存设置不等于就绪：报告实际验证结果和任何未完成的步骤。

## 命令

以下示例涵盖完整安装和可选附加组件。核心提供
`chat` 和 `ask_questions`；其他能力和资源命令需要其所属模块。用
`deeptutor module list` 查看已安装/已启用/就绪状态，用 `deeptutor plugin list` 查看已注册的能力和工具。更改模块前先停止
DeepTutor。源码检出使用 `module enable/disable`
和 `python scripts/install_extras.py <extra>` 安装可选依赖；
wheel 安装还支持 `module install/uninstall`。本地构建时，
向 `module install` 传入 `--wheelhouse /path/to/wheels`（见[模块](docs-for-dev/MODULES.md)）。

### 对话与能力

```bash
# 交互式 REPL
deeptutor chat
deeptutor chat --capability deep_solve --kb my-kb --tool web_search

# 一次性执行能力
deeptutor run chat "Explain Fourier transform"
deeptutor run deep_solve "Solve x^2 = 4" --kb textbook
deeptutor run deep_question "Linear algebra" --config num_questions=5
deeptutor run deep_research "Attention mechanisms" --kb papers --config mode=report --config depth=standard
deeptutor run visualize "Plot the unit circle"
deeptutor run math_animator "Visualize a Fourier series"

# `run` / `chat -c` 接受的能力：
# 用 `deeptutor plugin list` 发现当前能力和工具可用性。

# `run` 的选项：
#   --session <id>         恢复现有会话
#   --tool/-t <name>       启用工具（可重复）
#   --kb <name>            知识库（可重复）
#   --notebook-ref <ref>   笔记本引用，"<notebook_id>:<rec1>,<rec2>"（可重复）
#   --history-ref <id>     引用的会话 id（可重复）
#   --language/-l <code>   响应语言（默认：en）
#   --config <key=value>   能力配置（可重复）
#   --config-json <json>   能力配置（JSON 格式）
#   --format/-f <fmt>      输出格式：rich | json（默认：rich）
```

`deeptutor chat` 接受相同的 `--session / --tool / --kb / --notebook-ref / --history-ref / --language / --config / --config-json` 选项，外加 `--capability/-c <name>` 设置初始能力。

**`--tool` / `-t` 的工具**：用户可切换的工具为 `brainstorm`、`web_search`、`paper_search`、`zotero_search`、`reason`、`geogebra_analysis`、`imagegen` 和 `videogen`。上下文门控工具（`rag`、`exec`、记忆、笔记本等）按上下文/能力规则挂载；`--tool` 不会绕过这些规则。用 `--kb` 挂载知识库以启用 RAG；Knowledge Frontier 还需要研究模块。运行 `deeptutor plugin list` 查看已注册集合。

### 知识库

```bash
deeptutor kb list [--format rich|json]              # 列出所有知识库
deeptutor kb info <name>                            # 显示知识库详情（JSON）
deeptutor kb create <name> --doc file.pdf           # 从文档创建（--doc/-d 可重复）
deeptutor kb create <name> --docs-dir ./papers      # ...或从文档目录创建
deeptutor kb add <name> --doc more.pdf              # 增量添加文档
deeptutor kb search <name> "query text" [--mode hybrid] [--format rich|json]
deeptutor kb eval <name> --dataset qa.jsonl [--top-k 5] [--mode hybrid] [--save report.json]
#   针对 QA 集评估检索质量（JSONL：{"query": ..., "gold": [...]}）；
#   报告每个用例及平均的 Recall@k / Precision@k / nDCG@k / MRR / MAP / Hit@k。
deeptutor kb set-default <name>                     # 设为默认知识库
deeptutor kb delete <name> [--force]                # 删除知识库
```

### 伙伴

伙伴是 IM 连接的学习伴侣（原“TutorBot”）。

```bash
deeptutor partner list                              # 列出所有伙伴
deeptutor partner create <id> -n "My Tutor"         # 创建并启动新伙伴
#   -n/--name <text>   显示名称
#   -s/--soul <md>     Soul markdown（人设）
#   -m/--model <id>    模型覆盖
deeptutor partner start <id>                        # 启动伙伴
deeptutor partner stop <id>                         # 停止运行中的伙伴
```

### 技能

安装和管理技能，包括来自 EduHub 和 ClawHub 的包。
Hub 引用格式为 `<hub>:<slug>[@version]`（hub 前缀默认为 `eduhub`）。

```bash
deeptutor skill search "flashcards" [--hub clawhub] [--limit 10]
deeptutor skill install clawhub:some-skill[@1.2.0] [--name local-name] [--force] [--allow-unverified]
deeptutor skill list                                # 列出本地技能（含 Hub 来源）
deeptutor skill remove <name>                       # 移除用户层技能
```

### 书籍

BookEngine 的维护命令（编写/阅读通过 Web 应用进行）。

```bash
deeptutor book list                                 # 列出所有书籍（标记过期页面）
deeptutor book health <book_id>                     # 检查知识库漂移 + log.md 健康状态
deeptutor book refresh-fingerprints <book_id>       # 重新快照知识库指纹
```

### 记忆

```bash
deeptutor memory show [<target>]    # target：L3（所有全局文档，默认）| L2（所有表面）| 文档名（如 profile、chat）
deeptutor memory clear [<target>]   # target：all（默认）| trace（所有 L1）| 表面名（清除该表面的 L1）
#   --force/-f   跳过确认
```

### 会话

```bash
deeptutor session list [--limit 20]                 # 列出会话
deeptutor session show <id> [--format rich|json]    # 查看会话消息
deeptutor session open <id>                         # 在 REPL 中恢复会话
deeptutor session rename <id> --title "..."         # 重命名会话
deeptutor session delete <id>                       # 删除会话
```

### 笔记本

```bash
deeptutor notebook list                             # 列出笔记本
deeptutor notebook create <name> [--description "..."]
deeptutor notebook show <notebook_id> [--format rich|json]
deeptutor notebook add-md <notebook_id> <file.md> [--title "..."] [--type chat|question|research|solve]
deeptutor notebook replace-md <notebook_id> <record_id> <file.md>
deeptutor notebook remove-record <notebook_id> <record_id>
```

### 提供商

```bash
deeptutor provider login openai-codex               # OpenAI Codex 的 OAuth 登录
deeptutor provider login github-copilot             # 通过 GitHub 设备流登录
deeptutor provider login codebuddy                  # 验证 CodeBuddy SDK 登录 / 打开登录
```

### 系统

```bash
deeptutor config show [--home <path>]               # 打印解析后的配置（已脱敏）
deeptutor config providers                          # 支持的安装提供商（JSON）
deeptutor config apply <file.json> [--check] [--home <path>] # 应用/校验 setup JSON
deeptutor plugin list                               # 列出已注册的工具和能力
deeptutor plugin info <name>                         # 显示工具/能力的 schema + 可用性
deeptutor module list                               # 检查已安装 / 已启用 / 就绪状态
deeptutor serve [--host 0.0.0.0] [--port 8001] [--reload]   # 启动 API 服务器
deeptutor start [--home <path>] [--detach] [--no-browser] # 启动后端 + 前端
deeptutor stop [--home <path>]                      # 停止分离式启动器
deeptutor init [--cli] [--home <path>] [--non-interactive] # 向导或缺失默认值
deeptutor doctor [--online] [--format json]         # 本地就绪 / 可选实时 LLM 探测
```

## REPL 斜杠命令

在 `deeptutor chat` 内使用：

| 命令 | 效果 |
|:---|:---|
| `/quit` | 退出 REPL |
| `/session` | 显示当前会话 id |
| `/status` | 打印当前 REPL 状态 |
| `/new` 或 `/clear` | 启动新会话上下文 |
| `/regenerate` 或 `/retry` | 重新运行上一条用户消息 |
| `/tool on\|off <name>` | 切换工具 |
| `/cap <name>` | 切换能力 |
| `/kb <name>\|none` | 设置或清除知识库 |
| `/history add <id>` / `/history clear` | 管理历史引用 |
| `/notebook add <ref>` / `/notebook clear` | 管理笔记本引用 |
| `/show last\|<n>` | 展开捕获的工具结果或思考块 |
| `/refs` | 显示所有活动引用 |
| `/config show\|set\|clear` | 管理能力配置 |

## 典型工作流

**首次源码安装（在安装指南中准备好环境后）：**
```bash
cd DeepTutor
python -m pip install -r requirements/full.txt
python -m pip install -e . --no-deps
python -m pip check
deeptutor module list
deeptutor init        # 交互式引导安装（加 --cli 仅 CLI）
```

Web UI 需安装 Node.js 22 LTS，并在启动前于 `web/` 下运行 `npm ci --legacy-peer-deps`。仅 CLI 用户可跳过前端安装。遵循
安装指南的就绪检查；仅初始化不能验证模型。

**日常学习：**
```bash
deeptutor chat --kb textbook --tool web_search
```

**从文档构建知识库：**
```bash
deeptutor kb create physics --doc ch1.pdf --doc ch2.pdf
deeptutor run chat "Explain Newton's third law" --kb physics
```

**生成题目：**
```bash
deeptutor run deep_question "Thermodynamics" --kb physics --config num_questions=5
```

**本地运行完整 Web 应用：**
```bash
deeptutor start       # 后端 + 前端；Ctrl+C 停止
```