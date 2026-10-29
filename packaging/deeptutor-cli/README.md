# deeptutor-cli

仅 CLI 的 DeepTutor 发行版。它安装 `deeptutor` 命令，以及终端工作流、RAG、文档解析和模型提供商集成所需的 Python 模块，但不包含打包的 Next.js Web 资源，也不包含 `deeptutor start` 使用的 FastAPI/Uvicorn 服务器依赖。

如需本地仅 CLI 环境，请在仓库根目录安装：

```bash
python3 -m venv .venv-cli
source .venv-cli/bin/activate
python -m pip install --upgrade pip
python -m pip install -e ./packaging/deeptutor-cli
```

安装后请保留检出的代码，因为可编辑安装会将 `deeptutor` 命令指向这些源文件。