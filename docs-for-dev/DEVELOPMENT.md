# 开发 DeepTutor

[手册](README.md) · [架构](ARCHITECTURE.md) · [测试](TESTING.md)

## 准备源码环境

按照 [Agent 设置](AGENT_SETUP.md) 配置 Python 3.11–3.14、Node.js 22 LTS、提供商配置和就绪检查。源码检出会暴露所有模块；在安装可编辑核心之前，先安装其外部依赖：

```bash
python -m pip install -r requirements/full.txt
python -m pip install -e . --no-deps
python scripts/install_extras.py dev
cd web
npm ci --legacy-peer-deps
cd ..
```

使用 `scripts/install_extras.py` 安装可选的源码依赖。在可编辑检出之上安装功能 wheel 会模糊源码归属；请使用独立环境进行 [wheel 生命周期测试](MODULES.md#validation)。FFmpeg 和 LaTeX 等系统可执行文件仍是需要它们的功能的独立前置条件。

更新前检查 `git status`、当前分支和远程。基于 `dev` 开发；PR 目标遵循[贡献政策](../CONTRIBUTING.md)。复用合适的任务检出，或在并发工作需要隔离时使用独立工作树。保留无关编辑。运行时数据和测试输出不要纳入 Git。

## 使用专用运行时主目录

`DEEPTUTOR_HOME` 是拥有 `data/` 的目录，而非 `data/` 本身。选择一个明确的私有开发主目录，并在每条命令中使用它。例如，在新 shell 中：

```bash
export DEEPTUTOR_HOME="$HOME/.local/share/deeptutor-dev"
deeptutor init --non-interactive
deeptutor config show
deeptutor module list
```

不要将测试夹具指向现有学习者的运行时。内容工作区是独立的存储和授权范围；参见[工作区](workspaces.md)。应用不会自动加载项目的 `.env` 文件。

## 运行与检查

组合启动器使用 `deeptutor start`，并读取其输出中的实际 URL。分离开发进程时，在两个终端中使用相同的运行时主目录分别运行 API 和前端：

```bash
# 终端 1：API，带重载
deeptutor serve --host 127.0.0.1 --port 8001 --reload
```

```bash
# 终端 2：前端，从 web/ 目录运行
DEEPTUTOR_API_BASE_URL=http://127.0.0.1:8001 npm run dev
```

选择空闲端口；不要停止无关服务。确认浏览器请求通过前端代理到达预期 API。仅前端在监听并不能证明提供商、模块或回合就绪。使用 `deeptutor doctor --format json` 和原始用户操作进行验证。

## 在归属处修改

1. 用最小合成夹具复现行为。确定归属是适配器、共享运行时、能力、工具、功能存储、提供商还是前端投影。
2. 阅读最近的实现及其测试。对于 API 变更，沿请求验证追踪认证、账户/工作区解析、持久化、事件和 UI 消费。
3. 修改该归属，而不是新增第二个运行时、路径解析器、提供商客户端或存储抽象。共享协议保持轻量导入。
4. 受影响时更新模块声明、请求契约、提示词和语言环境字符串。用其归属脚本重新生成派生文件。
5. 运行[适合该变更的检查](TESTING.md)，复现原始操作，并在 PR 中说明任何验证限制。

子进程使用显式参数列表和 `shell=False`；取消时保留进程树清理。通过现有账户和工作区上下文解析存储，而不是构造全局数据路径。切勿向浏览器响应返回凭据或私有提供商重放元数据。

## 生成文件与文档

仓库有意跟踪生成的 API 契约、需求文件和依赖锁。更新其生成器或权威输入，然后重新生成。`web/.next*`、已安装依赖、字节码、日志和手动验收产物保持不跟踪。

公共行为变更时，更新相关手册主题和 README 链接。将可复用的可执行示例放在本手册的 [`examples/`](examples/learning_echo/README.md) 下。命令示例应可从仓库根目录使用，并避免个人机器路径。