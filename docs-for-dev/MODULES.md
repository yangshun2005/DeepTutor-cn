# 模块与安装

[手册](README.md) · [相关指南](TESTING.md)

DeepTutor 有一个源码仓库和多个互不相交的 Python 发行包。`deeptutor` 拥有共享的轮次运行时、CLI 和 Python SDK。功能发行包将代码加入同一包命名空间，不覆盖核心文件。`deeptutor-cli` 是兼容性元包：它依赖核心，自身不含任何 Python 文件或控制台脚本。`deeptutor[full]` 是安装配置，不是永久元包——永久元包会妨碍单独移除各个应用。

模块化发行包系列从 1.7.0 开始，每次发布必须一起发布。1.6.15 及更早的包使用旧布局，没有这些边界。也支持本地 wheel 构建。

## 源码检出

```sh
python -m pip install -r requirements/full.txt
python -m pip install -e . --no-deps
(cd web && npm ci --legacy-peer-deps)
deeptutor module list
deeptutor module disable solve
deeptutor module enable solve
```

在源码检出中，所有源码模块都已存在。`enable` 和 `disable` 执行与 wheel 安装相同的组装规则。`install` 和 `uninstall` 刻意要求 wheel 环境，因此绝不会删除受版本控制的源码文件。源码依赖文件由 `python scripts/sync_module_requirements.py` 生成。在 CI 中使用 `--check` 检测漂移。`requirements/core.txt` 和 `server.txt` 是最小集；`full.txt` 提供常规的完整源码/Docker 运行时。源码归档和 Docker 镜像通过项目文件和模块构建脚本识别，无需 `.git`。可选源码依赖可用 `python scripts/install_extras.py <extra>` 添加；它解析模块依赖，不会在源码树上安装重复的功能 wheel。

## 独立 wheel 安装

```sh
python scripts/build_modules.py --output dist/modules
python -m venv /tmp/deeptutor-modules
/tmp/deeptutor-modules/bin/python -m pip install --find-links dist/modules deeptutor
/tmp/deeptutor-modules/bin/deeptutor module install solve --wheelhouse dist/modules
/tmp/deeptutor-modules/bin/deeptutor module install renderer-manim --wheelhouse dist/modules
/tmp/deeptutor-modules/bin/deeptutor module disable solve
/tmp/deeptutor-modules/bin/deeptutor module enable solve
/tmp/deeptutor-modules/bin/deeptutor module uninstall solve --dry-run
/tmp/deeptutor-modules/bin/deeptutor module uninstall solve
```

Windows 上使用对应的 `Scripts/python.exe` 和 `Scripts/deeptutor.exe`。运行命令时使用与应用相同的 `DEEPTUTOR_HOME`。Python 包按环境安装；启用状态保存在 `<home>/data/system/modules.json`，作用于该部署，而非单个学习者。`--wheelhouse` 用本地构建的 wheel 补充包索引。

先安装 `server` 以提供 API，再安装 `web` 以提供打包的 Web 资源。Web wheel 需要 `scripts/prepare_web_package.py` 生成的资源才能用于 `deeptutor start`；源码开发服务器使用检出中的 `web/` 目录。构建和发布自动化在构建 wheel 系列之前准备 Web 资源。

## 从 1.6.15 升级

停止旧进程并备份其运行时主目录。先构建带打包 Web 资源的候选 wheel。在旧环境的一次性副本中，用 `python -m pip install --upgrade --find-links dist/modules "deeptutor[full]==<candidate-version>"` 升级。如果安装了旧版 `deeptutor-cli`，先用 `python -m pip uninstall deeptutor-cli` 卸载，再运行升级命令，可选地加上新的兼容别名。旧版 CLI 与新核心拥有相同的 Python 文件：pip 会先安装新的核心依赖，再移除旧 CLI，而移除旧 CLI 会删除新文件。先移除旧 CLI 可避免此冲突，并保留运行时主目录数据。若仅迁移 CLI，在该卸载之后安装 `deeptutor-cli==<candidate-version>`。使用确切的候选版本，避免解析到更早的已发布版本。保持 `DEEPTUTOR_HOME` 不变，运行 `pip check`，检查 `deeptutor module list`，并在升级真实环境前验证聊天、持久化会话和 Web 启动。切勿针对用户数据的唯一副本进行测试。仅升级核心不会保留旧的完整应用配置。

`deeptutor/__version__.py` 是所有构建的版本来源，包括源码 CLI 兼容包。extras 是 `tool.deeptutor.optional-dependencies` 中的模板；setuptools 和模块构建器都会将其内部固定版本解析为该版本。发布版本号变更不得让依赖仍指向上一模块系列。

## 归属与组装

`deeptutor/modules/manifests/*.json` 是权威目录。每个清单拥有路径、依赖要求、轮次能力、循环扩展、工具名、API 路由与授权策略、CLI 组、导航路径和后台服务回调。最长匹配路径优先。共享协议和轻量上下文元数据留在核心；功能实现留在各自的发行包中。构建器拒绝重叠的 wheel 文件。

| 模块系列 | 内容 |
| --- | --- |
| core | 持久化轮次、聊天、提问、共享工具、CLI/SDK、设置和会话 |
| server / web | API 与认证适配器 / 打包的 Next.js 资源 |
| solve / questions / research | 规划循环 / 问题生成 / 研究流水线 |
| visualize / renderer-manim | HTML/SVG/图表可视化器 / 可选 Manim 流水线 |
| video-lesson / audio-overview | 浏览器课程 / 音频概览生成 |
| knowledge / parsing | 检索与摄取 / 文档解析实现 |
| reading / mastery / books | 阅读与观看 / 引导式学习 / 书籍应用 |
| memory / notebook / co-writer / calendar | 可选应用服务及其路由 |
| partners / providers | 伴侣运行时与通道 SDK / 可选模型提供方 SDK |
| parser-* | 额外的解析器依赖发行包，需要 parsing |

依赖显式且无环。例如，reading 目前依赖 knowledge、parsing、mastery 和 notebook，因为其应用使用这些存储。books 还依赖 questions、visualize 和 mastery，用于标准章节块和选定来源的接地。可选集成单独列出：Calendar 无需 Mastery 即可工作，Research 和 Questions 无需 Knowledge 即可工作，Parsing 从不导入 knowledge 存储。选择不可用的集成会返回安装提示。Knowledge Frontier 同时需要 Research 和 Knowledge；工具贡献可以声明额外要求，而不会强制这些模块进入其所属应用的每次使用。禁用依赖会被阻止，直到其已启用的依赖方被禁用。卸载依赖会被阻止，只要仍有已安装的依赖方存在，包括已禁用的依赖方。启用/安装按顺序解析依赖。核心不可禁用或移除。

加载器不会导入缺失或已禁用的能力/循环实现。请求 Schema 是惰性的。循环扩展每轮实例化一次，激活状态随上下文变化重新评估；实例从不跨轮共享。托管插件的撤销仍会检查。后台启动/停止回调保持配对，以支持部分启动回滚。现有 API 认证策略保留在贡献的路由上；模块状态不是权限授予。

`GET /api/modules` 需要认证，并分别暴露已安装、已启用和就绪状态。运行时设置页面显示这些状态和离线命令。侧边栏和页面访问使用导航归属；API 路由组装也会排除缺失模块，包括共享路由中的功能特定路径。就绪检查检查包要求、核心/模块版本匹配和外部可执行文件。已配置的提供方凭据和服务连通性仍由现有设置就绪系统处理。

## 生命周期保证

官方模块共享一个解释器。运行中的应用容器持有共享环境租约；模块变更需要跨进程的独占租约。第二个租约在不同 Python 环境共享同一主目录时保护部署选择。在更改包或选择之前，停止 CLI 轮次和 API worker。直接的外部 `pip` 命令无法被此建议性锁阻止。

安装先解析依赖，拒绝替换现有包版本，并在修改环境前构建确切的已解析构建产物。如果添加包后安装失败，这些新添加的发行包会被移除，包括保存选择失败的情况。格式错误的选择在任何包变更之前被拒绝。选择仅在包操作成功后原子替换。依赖冲突请使用新环境或显式环境升级。这不是跨机器崩溃的原子文件系统事务；标准 pip 修复/重装仍是机器中断后的恢复路径。

卸载仅移除选定的发行包。共享依赖会保留，因为其他应用可能使用它们。用户设置、会话、笔记本、知识库、生成的构建产物和凭据绝不会被清除。重装使用保留的数据。没有隐式的数据删除命令。

从运行中的设置/安装会话安装解析器包现在返回离线模块命令。它不再在活动轮次下升级包。模型权重下载仍是后台任务，因为它们不改变 Python 代码。第三方托管插件保留其现有的隔离环境。

## 第三方 worker 扩展

托管插件 worker 使用自己的环境和生命周期，与官方模块 wheel 分开。参见[插件流式传输与取消](PLUGINS.md#worker-protocol-and-cancellation)和[可运行示例](examples/learning_echo/README.md)。

## 验证

```sh
python scripts/sync_module_requirements.py --check
python -m pytest tests/modules tests/test_packaging_metadata.py
python scripts/build_modules.py --output dist/modules
python scripts/check_module_install.py --wheelhouse dist/modules --extended
```

安装冒烟测试创建一个一次性虚拟环境，使用本地模拟模型（无提供方凭据），检查核心聊天和持久化重放，安装 Solve，运行它，卸载它，验证持久化文件，再次检查核心，然后重装 Solve。扩展模式安装 Parsing、Research 和 Calendar（不含 Knowledge 或 Mastery），验证解析及其缓存以及独立的研究和日历路径，然后逐个移除模块，同时验证 Solve 和保留的数据。源码隔离测试还禁止在 25 个模块各自的新子进程中存在未声明的导入，并组装它们的贡献。Manim 额外需要系统渲染库、FFmpeg 和 LaTeX。对其渲染器的更改应附带渲染和取消测试。Web 应用仍是一个前端发行包；可独立替换的第三方前端包不在本 Python 模块契约范围内。

## 实现与验证

- [deeptutor/modules/manifests/](../deeptutor/modules/manifests/)
- [scripts/build_modules.py](../scripts/build_modules.py)
- [scripts/check_module_install.py](../scripts/check_module_install.py)