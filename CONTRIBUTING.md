# 为 DeepTutor 做贡献

感谢你有兴趣为 DeepTutor 做贡献！我们欢迎各技术水平的开发者，共同打造下一代智能学习伙伴。

<p align="center">
<a href="https://discord.gg/eRsjPgMU4t"><img src="https://img.shields.io/badge/Discord-Join_Community-5865F2?style=for-the-badge&logo=discord&logoColor=white" alt="Discord"></a>&nbsp;
<a href="https://github.com/HKUDS/DeepTutor/issues/78"><img src="https://img.shields.io/badge/WeChat-Join_Group-07C160?style=for-the-badge&logo=wechat&logoColor=white" alt="WeChat"></a>&nbsp;
<a href="./Communication.md"><img src="https://img.shields.io/badge/Feishu-Join_Group-00D4AA?style=for-the-badge&logo=feishu&logoColor=white" alt="Feishu"></a>
</p>

---

## 目录

- [维护者](#维护者)
- [分支策略](#分支策略)
- [贡献者快速上手](#贡献者快速上手)
- [开发环境搭建](#开发环境搭建)
- [代码质量与安全](#代码质量与安全)
- [编码规范](#编码规范)
- [提交信息格式](#提交信息格式)
- [安全最佳实践](#安全最佳实践)

---

## 维护者

[@pancacake](https://github.com/pancacake) — 目前只有我一人！

---

## 分支策略

我们采用多分支模型来保持开发有序：

| 分支 | 用途 | 稳定性 |
|---|---|---|
| `dev` | 通用开发 | 可能存在 Bug 或破坏性变更 |
| `multi-user` | 多用户场景开发 | 实验性，聚焦多租户功能 |

> [!IMPORTANT]
> 请**不要**直接向 `main` 提交 PR。所有贡献都应指向 `dev` 或 `multi-user`。

### 我应该以哪个分支为目标？

如果你的 PR 包含以下内容，**目标为 `dev`**：

- 新功能或新特性
- 可能影响现有行为的重构
- API 或配置变更
- 通用 Bug 修复

如果你的 PR 包含以下内容，**目标为 `multi-user`**：

- 多用户 / 多租户相关功能
- 会话隔离、用户管理或权限变更
- 协作或共享工作区功能

> [!NOTE]
> 拿不准时，目标为 `dev` —— 它是默认开发分支。

---

## 贡献者快速上手

1. **Fork 并克隆**仓库。
2. 开始前先与目标分支**同步**：

```bash
git checkout dev && git pull origin dev
```

3. 从目标分支**创建**功能分支：

```bash
git checkout -b feature/your-feature-name
```

4. **开发**你的改动，遵循下方编码规范。
5. 运行 pre-commit 检查进行**验证**：

```bash
pre-commit run --all-files
```

6. 向正确的目标分支**提交** Pull Request（除非是热修复或仅文档改动，否则不要提交到 `main`）。

> [!TIP]
> 浏览我们的 [Issues](https://github.com/HKUDS/DeepTutor/issues)，寻找标记为 `good first issue` 的任务作为起点。在 Issue 下留言，告知他人你正在处理。

---

## 开发环境搭建

请从[开发者与 Agent 手册](docs-for-dev/README.md)开始。
[Agent 搭建](docs-for-dev/AGENT_SETUP.md)负责安装与配置；
[开发](docs-for-dev/DEVELOPMENT.md)与[测试](docs-for-dev/TESTING.md)
说明源码工作流与验证关卡。

<details>
<summary><b>搭建你的环境</b></summary>

**第 1 步：创建虚拟环境**

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

**第 2 步：安装依赖**

```bash
python -m pip install -r requirements/full.txt
python -m pip install -e . --no-deps
python scripts/install_extras.py dev
cd web
npm ci --legacy-peer-deps
cd ..
```

</details>

<details>
<summary><b>设置 Pre-commit（仅首次）</b></summary>

**第 1 步：安装 pre-commit**

```bash
pip install pre-commit
# Or: conda install -c conda-forge pre-commit
```

**第 2 步：安装 Git 钩子**

```bash
pre-commit install
```

**第 3 步：初始化密钥基线**

如果遇到误报的密钥（如 API 哈希占位符），更新基线：

```bash
detect-secrets scan > .secrets.baseline
```

</details>

### 常用命令

| 任务 | 命令 |
|---|---|
| 检查工作区干净 + 已跟踪文件卫生 | `python3 scripts/check_workspace_hygiene.py` |
| 检查仓库卫生 | `python3 scripts/check_repo_hygiene.py` |
| 检查所有文件 | `pre-commit run --all-files` |
| 静默检查 | `pre-commit run --all-files -q` |
| 更新工具 | `pre-commit autoupdate` |
| 紧急跳过 | `git commit --no-verify -m "message"` *（不推荐）* |

### 生成文件与工作树

构建产物不要纳入 Git。`web/.next*`、`node_modules`、测试报告和
字节码缓存均可重新生成，必须保持未跟踪状态。如果某个构建产物已被跟踪，
用 `git rm --cached` 将其从索引中移除，而不是删除应用运行所需的本地文件。

全新检出的仓库可通过以下命令启用无依赖的安全钩子：

```bash
git config core.hooksPath scripts/hooks
```

该钩子还会阻止在 `main` 上意外直接提交。确需在本地 `main` 提交的发布维护者，
可一次性启用 `git config deeptutor.allowMainCommit true`，随后立即移除该设置。

为每个功能使用独立的 Git 工作树（`git worktree add ../DeepTutor-<task>
-b <branch> dev`），并保持主检出干净。这样构建、测试和长时间运行的 Agent
可独立运行，互不覆盖输出。移除工作树前，先提交或显式保留其改动；
不要将 `git reset --hard` 或 `git clean` 当作日常清理捷径。
---

## 代码质量与安全

我们使用自动化工具（通过 `pyproject.toml` 和 `.pre-commit-config.yaml` 配置）维持高标准：

| 工具 | 用途 |
|---|---|
| **Ruff** | Python 代码检查与格式化 |
| **Prettier** | 前端与配置文件格式化 |
| **detect-secrets** | 硬编码密钥扫描 |
| **pip-audit** | 依赖漏洞扫描 |
| **Bandit** | 安全问题分析 |
| **MyPy** | 静态类型检查 |
| **Interrogate** | 文档字符串覆盖率报告 |

> [!IMPORTANT]
> 本地 pre-commit 钩子可能只显示警告，但 **CI 会执行严格检查**，并自动拒绝未通过的 PR。

---

## 编码规范

### Python

- 所有函数签名使用**类型提示**。
- 字符串格式化优先使用 **f-string**。
- 遵循 **PEP 8**（由 Ruff 强制执行）。
- 函数保持**小而专注**，只承担单一职责。

### 文档

- 每个新模块、类和公共函数都应有**文档字符串**（Google Python Style Guide 格式）。
- 如果你的改动引入新功能或配置，请更新 `README.md`。

---

## 提交信息格式

```
<type>: <short description>

[optional body]
```

| 类型 | 说明 |
|---|---|
| `feat` | 新功能（MINOR 版本号递增） |
| `fix` | Bug 修复（PATCH 版本号递增） |
| `docs` | 仅文档改动 |
| `style` | 格式化，无逻辑改动 |
| `refactor` | 代码重构，无新功能或修复 |
| `test` | 新增或修正测试 |
| `chore` | 构建流程、工具或依赖更新 |

---

## 安全最佳实践

### 文件上传

- **大小限制**：普通文件上限 100 MB；PDF 上限 50 MB。
- **校验**：多层校验（扩展名 + MIME 类型 + 内容净化）。
- **净化**：所有文件名均经净化，防止路径遍历。

### 开发规范

- **子进程**：始终使用 `shell=False`，防止命令注入。
- **路径**：使用 `pathlib.Path` 以保证跨平台兼容。
- **行尾符**：关键脚本通过 `.gitattributes` 强制使用 LF（Unix）行尾符。

---

有疑问？在 [Discord](https://discord.gg/eRsjPgMU4t) 联系我们。让我们一起构建 AI 辅导的未来！