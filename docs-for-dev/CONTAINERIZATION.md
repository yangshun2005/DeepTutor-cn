# DeepTutor 容器化

[手册](README.md) · [Agent 设置](AGENT_SETUP.md) · [存储](DATA_STORAGE.md)

本文介绍如何从容器镜像部署 DeepTutor：推荐的 `docker run` 方式、加固的无根 Podman 方式（只读根文件系统）、运行时配置、可选的 PocketBase sidecar，以及默认安全姿态背后的安全说明。

PyPI / 源码安装请参阅主 [README.md](../README.md)。本文只讲如何运行已发布的镜像。

---

## 概述

已发布的 `ghcr.io/hkuds/deeptutor` 镜像基于 `python:3.11-slim`，在单个容器内通过 `supervisord` 同时运行 FastAPI 后端（`:8001`）和 Next.js 前端（`:3782`）。私有运行时目录（容器内 `/app/data`）存放设置、凭据、数据库、记忆、知识库和日志。默认内容工作区也位于此处，但也可以单独挂载到 `/workspace`。请绑定挂载运行时目录和单独的内容工作区，使两者在重启后都能保留。

镜像支持三种部署形态：

1. **`docker run`** — 简单方式。有根、可写根文件系统，在 `/app/data` 上单一绑定挂载。
2. **`docker compose`**（`docker-compose.yml`）— 同一镜像，外加 PocketBase sidecar 和沙箱运行器 sidecar。仍有根、可写根文件系统。
3. **`podman compose -f compose.yaml`** — 加固方式。无根（`userns_mode: keep-id`）、只读根文件系统、用 tmpfs 替代可写系统目录、在 `./data` 上绑定挂载。

使形态 3 可行的架构改动是：URL 知识不再存在于前端 bundle 中。具体来说：

- bundle 构建时不含 `NEXT_PUBLIC_API_BASE` 占位符，也不对构建产物执行 `sed -i`。`web/lib/api.ts` 将 `apiUrl` 和 `wsUrl` 导出为单行透传，因此浏览器通过前端（`:3782/api/...`）请求相对路径。
- `web/proxy.ts` 捕获 `/api/*` 和 `/ws/*`，在请求时将其重写到 `DEEPTUTOR_API_BASE_URL`。该环境变量由容器入口点在每次启动时从 `data/user/settings/system.json` 读取并设置。
- `start-frontend.sh` 设置 `PORT`/`HOSTNAME` 并 `exec` `node /app/web/server.js`。不修改 bundle。
- `supervisord` 以 root（PID 1）运行，并通过每个程序的 `user=` 指令将各程序（后端、前端）降权到非 root 的 `deeptutor` 用户（UID 1000），因此应用进程保持非 root。在主机上使用 `userns_mode: keep-id` 时，该 UID 映射到你的主机 UID；使用普通 `docker run` 时，它是容器内的普通非特权用户。

以下是完整的逐安装指南。

---

## Docker（默认）

最简单的部署方式。一个容器、一个私有数据卷、一个端口映射。

```bash
docker run --rm --name deeptutor \
  -p 127.0.0.1:3782:3782 \
  -v deeptutor-data:/app/data \
  ghcr.io/hkuds/deeptutor:latest
```

打开 <http://127.0.0.1:3782>。容器首次启动时创建 `/app/data/user/settings/*.json`；在 Web 设置页面配置模型提供商。配置、API 密钥、日志、工作区文件、记忆和知识库持久化在 `deeptutor-data` 命名卷中。

### Unraid / NAS 绑定挂载（PUID / PGID）

Unraid 和许多 NAS 模板将 appdata 归属于一个**不是** UID 1000 的主机用户（通常是 `nobody:users` = 99:100，或某个命名共享用户）。这些主机还会在重启时还原容器内的 `chown`。镜像仍会将后端/前端降权到非 root 的 `deeptutor` 用户；入口点会将该用户的 UID/GID 重映射以匹配卷：

```bash
docker run --rm --name deeptutor \
  -e PUID=99 -e PGID=100 \
  -p 127.0.0.1:3782:3782 \
  -v /mnt/user/appdata/deeptutor:/app/data \
  ghcr.io/hkuds/deeptutor:latest
```

如果运行时用户仍无法写入 `/app/data`，入口点会**退出**并给出所有者/权限模式消息，而不是启动后稍后报告 `Knowledge base not initialized (llamaindex)`。无根 Podman 跳过重映射（缺少 `CAP_SETUID`）并保留当前用户。`PUID=0` / `PGID=0` 会被拒绝，确保应用绝不以 root 运行。

### 选择主机内容工作区

应用数据挂载包含私有设置、凭据、数据库、记忆和其他运行时状态。请将其与 Agent 可读取的**内容工作区**分开。要暴露主机文件夹，将其挂载到稳定的 `/workspace` 路径并将部署锁定到该路径：

```bash
mkdir -p "$PWD/deeptutor-workspace/outputs"
docker run --rm --name deeptutor \
  -p 127.0.0.1:3782:3782 \
  -v deeptutor-data:/app/data \
  -v "$PWD/deeptutor-workspace:/workspace" \
  -e DEEPTUTOR_WORKSPACE_ROOT=/workspace \
  -e DEEPTUTOR_WORKSPACE_ALLOWED_ROOTS=/workspace \
  ghcr.io/hkuds/deeptutor:latest
```

对于提供的 Compose 文件：

```bash
DEEPTUTOR_WORKSPACE_HOST=/absolute/host/folder \
  python scripts/docker_compose.py -f docker-compose.yml up -d
```

该辅助脚本在 Compose 启动前创建主机文件夹及其嵌套的 `outputs/` 目录，避免 root 拥有的绑定挂载。如果省略该变量，则使用 `./data/user/workspace`。容器始终看到 `/workspace`，因此存储的设置和面向模型的路径在不同主机间保持可移植。这是启动/部署选择，在**设置 → 工作区**中显示为已锁定。

使用 `docker-compose.yml` 时，沙箱运行器以只读方式接收内容树，外加 `/workspace/outputs` 的可写覆盖层；它不会接收 `/app/data`、用户设置或凭据。因此生成的程序只读取所选内容并仅写入回合范围内的输出。单容器和无根 Podman 形态使用该容器中可用的最佳隔离后端，并在工作区设置中报告有效级别。

注意：

- **只需发布 `3782`。** 浏览器只与前端源（`:3782`）通信；所有 `/api/*` 和 `/ws/*` 流量由 Next.js 中间件（`web/proxy.ts`）在请求时转发到**容器内**的 FastAPI 后端，该中间件读取 `DEEPTUTOR_API_BASE_URL`（默认 `http://localhost:8001`）。UI 正常工作**不需要**将 `:8001` 暴露到主机。发布 `:8001`（`-p 127.0.0.1:8001:8001`）是可选的——仅便于直接访问 API（curl、脚本）或调试。
- **不同主机端口：** 修改每个 `-p host:container` 映射的左侧（例如 `-p 127.0.0.1:8088:3782`）。如果在 `data/user/settings/system.json` 中修改容器侧端口（`backend_port`、`frontend_port`），请重启容器并更新每个映射的右侧以匹配。
- **后台运行：** 添加 `-d`，然后用 `docker logs -f deeptutor` 跟踪日志，`docker stop deeptutor` 停止，复用名称前先 `docker rm deeptutor`。`deeptutor-data` 卷在重启间保留私有运行时数据和默认内容工作区；单独挂载的内容工作区持久化在其主机路径。

### 临时本地 Codex OAuth 桥接

OpenAI Codex 会将浏览器重定向到固定回环端口 `1455` 或 `1457`。默认容器网络与主机回环隔离，因此仅在登录期间将这两个端口发布到 Web 前端。启动临时桥接前，两个主机端口都必须空闲。

对于 `docker run`，停止正常容器，临时用两个额外的仅回环映射重新运行同一镜像和数据卷：

```bash
docker run --rm --name deeptutor \
  -p 127.0.0.1:3782:3782 \
  -p 127.0.0.1:1455:3782 \
  -p 127.0.0.1:1457:3782 \
  -v deeptutor-data:/app/data \
  ghcr.io/hkuds/deeptutor:latest
```

对于 Compose，将相同的临时覆盖层添加到通常使用的基础文件：

```bash
# Source build with sidecars
python scripts/docker_compose.py \
  -f docker-compose.yml -f compose.codex-oauth.yaml \
  up -d --force-recreate deeptutor

# Pre-built GHCR image
python scripts/docker_compose.py \
  -f docker-compose.ghcr.yml -f compose.codex-oauth.yaml \
  up -d --force-recreate deeptutor

# Rootless Podman
podman compose -f compose.yaml -f compose.codex-oauth.yaml \
  up -d --force-recreate deeptutor
```

完成**设置 → 模型 → OpenAI Codex → 使用 Codex 登录**。状态变为**已连接**后，停止临时 `docker run` 容器并返回上面的正常命令。对于 Compose，重新运行同一基础命令但不带 `compose.codex-oauth.yaml`；保留 `--force-recreate deeptutor` 以移除临时端口绑定。

这会释放主机端口 `1455` 和 `1457`；凭据保留在持久的 `/app/data/system` 树中。将每个回调映射绑定到 `127.0.0.1`，绝不要在 LAN 或公共接口上暴露——该覆盖层在这两个端口上发布**整个**前端，而不仅仅是 `/auth/callback`。对于容器侧前端端口不是 `3782` 的手动 `docker run`，修改右侧的 `3782` 目标；`scripts/docker_compose.py` 会处理配置的自定义端口。对于单容器安装，如果模型生成的代码不得与机密共享容器信任边界，请将 `sandbox_allow_subprocess` 设为 `false`。

### 一次性迁移：`docker-compose.ghcr.yml` 现在挂载整个 `./data`

`docker-compose.ghcr.yml` 过去只绑定挂载三个子树（`data/user`、`data/memory`、`data/knowledge_bases`），因此其他所有内容——`data/system`（JWT 签名密钥、账户、授权、审计日志、每所有者 Codex 令牌）、`data/users`（每用户工作区）、`data/partners`、`data/cli-apps`——都位于容器的可写层，每次重建时被丢弃。现在它挂载整棵树，与 `docker-compose.yml` 和 `compose.yaml` 一致。

**升级后首次 `up -d` 之前**，从运行中的容器复制该状态，否则空的主机目录会遮蔽它，DeepTutor 会重新生成认证密钥（使所有人登出）并以无任何非管理员账户的状态启动：

```bash
for tree in system users partners cli-apps; do
  docker cp "deeptutor:/app/data/$tree" "./data/$tree" 2>/dev/null || true
done
```

使用命名卷（`-v deeptutor-data:/app/data`）或源码构建 Compose 文件的部署从未受影响——它们本就持久化整棵树。

### 远程 / 反向代理部署

对于常见的**单容器**场景（本镜像），你**完全不需要**配置 API 基址。浏览器针对提供 UI 的任意源（`https://deeptutor.example.com`）发出相对 `/api/*` 和 `/ws/*` 请求，容器内的 Next.js 中间件将其转发到 `localhost:8001` 上的后端。只需将反向代理 / TLS 终止器指向已发布的 `:3782` 即可。

只有在**分离部署**（后端运行在单独容器中）时才需要设置 API 基址。在主机上编辑 `data/user/settings/system.json`（位于 `deeptutor-data` 卷内——用 `docker volume inspect deeptutor-data` 查找其挂载点），设置前端容器用于访问后端容器的网络内地址：

```json
{
  "next_public_api_base": "http://backend:8001"
}
```

入口点在每次启动时读取此值并为 `proxy.ts` 导出 `DEEPTUTOR_API_BASE_URL`（优先级：`next_public_api_base`，然后 `next_public_api_base_external`，然后 `http://localhost:8001`）。注意，由于代理是**服务端**的，`DEEPTUTOR_API_BASE_URL` 是前端*服务器*用于访问后端的地址——不是浏览器看到的 URL。`public_api_base` 作为兼容别名被接受，并在保存时规范化为 `next_public_api_base_external`。

CORS 使用前端**源**，而非 API URL。禁用认证时，DeepTutor 默认允许正常的 HTTP/HTTPS 浏览器源。启用认证时，添加精确的前端源：

```json
{
  "cors_origins": ["https://deeptutor.example.com"]
}
```

### 主机 LLM 提供商（Ollama / LM Studio / llama.cpp / vLLM / Lemonade）

在 Docker 内，`localhost` 是容器本身，不是你的主机。要访问主机上运行的模型服务，使用主机网关（推荐）：

```bash
docker run --rm --name deeptutor \
  -p 127.0.0.1:3782:3782 -p 127.0.0.1:8001:8001 \
  --add-host=host.docker.internal:host-gateway \
  -v deeptutor-data:/app/data \
  ghcr.io/hkuds/deeptutor:latest
```

然后在**设置 → 模型**中，将提供商 Base URL 指向 `host.docker.internal`：

- Ollama LLM：`http://host.docker.internal:11434/v1`
- Ollama embedding：`http://host.docker.internal:11434/api/embed`
- LM Studio：`http://host.docker.internal:1234/v1`
- llama.cpp：`http://host.docker.internal:8080/v1`
- Lemonade：`http://host.docker.internal:13305/api/v1`

Docker Desktop（macOS/Windows）通常无需 `--add-host` 即可解析 `host.docker.internal`。在 Linux 上，该标志是可移植的方式。

**Linux 替代方案——主机网络：** 添加 `--network=host` 并去掉 `-p` 标志。容器直接共享主机网络，因此打开 <http://127.0.0.1:3782>（或 `system.json` 中的 `frontend_port`），主机服务可用普通 localhost URL 访问。

在主机网络模式下，进程直接绑定在主机接口上（没有 `-p 127.0.0.1:` 前缀来限定范围）。要使它们不暴露到 LAN，设置 `BACKEND_HOST=127.0.0.1` 和 `FRONTEND_HOST=127.0.0.1`——它们覆盖 uvicorn 的 `--host` 和 Next.js 的 `HOSTNAME`（两者默认 `0.0.0.0`）。仅在 `--network=host` 时使用这些：在桥接模式下绑定回环会破坏已发布的 `-p` 端口转发。

---

## Podman / 无根 / 只读根文件系统

对于想要最强默认姿态——无根、只读根文件系统——的用户，`compose.yaml` 是受支持的起点。它拉取相同的 `ghcr.io/hkuds/deeptutor:latest` 镜像，依赖入口点 chown + supervisord 的逐程序权限降级、URL 转发的 `proxy.ts` 以及主机侧绑定挂载来使其工作。

```bash
cp .env.example .env       # then edit if needed
podman compose -f compose.yaml up -d
podman compose -f compose.yaml ps
podman compose -f compose.yaml logs -f deeptutor
```

验证无根已激活（`podman info | grep -i rootless` 应报告 `true`）。

`compose.yaml` 做了什么，以及为什么：

- **每个服务上的 `read_only: true`。** 容器的根文件系统只读。唯一可写表面是每个服务列出的 `tmpfs:` 挂载加上绑定挂载的 `./data` 和内容工作区目录。
- **`userns_mode: keep-id`。** 使用运行时的 keep-id 用户命名空间。在目标主机上检查实际映射；不要假设容器 UID 0 或 UID 1000 对每个主机账户映射相同。`:U` 挂载选项会为容器映射调整所有权，可能改变主机目录所有权。使用专用数据目录并验证写访问。
- **为运行时预期写入的系统目录提供 `tmpfs:` 挂载。** `/tmp`（Python/Node 临时文件）、`/run` 和 `/var/run`（pidfile）、`/var/log`、`/root`、`/home`。大小故意宽松；按需裁剪。
- **无命名卷。** Podman 自动创建以 userns 映射 root（UID 100000）拥有的命名卷，因此 755 权限 + 错误所有者 = 首次 JSON 写入时 `PermissionError`。在你拥有的主机目录上绑定挂载可干净工作。
- **仅回环端口绑定。** 每个 `ports:` 条目都有 `127.0.0.1:` 前缀。去掉前缀以在所有接口上暴露。
- **无沙箱运行器 sidecar。** `docker-compose.yml` 形态包含一个加固 side