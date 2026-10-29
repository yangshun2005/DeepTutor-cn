# 可选的请求筛查（fastapi-guard）

[手册](README.md) · [相关指南](CONTAINERIZATION.md)

DeepTutor 可在后端内用 [fastapi-guard](https://github.com/Guard-Core/fastapi-guard) 筛查 HTTP 请求。该功能**默认关闭**。关闭时不会导入该包，中间件栈保持不变。与其他新设置一样，`system.json` 会在下次加载时补上默认关闭的 `security_guard` 块，格式错误的块会被替换为默认值。

该防护是应用层控制，与边缘的反向代理、WAF 或防火墙互补；即使后端端口被直接访问（例如公开的 `:8001`、局域网内的 `deeptutor serve` 或隧道），它仍然生效。

## 功能

对每个 HTTP 请求按以下顺序检查：

- **IP 列表**：可选的白名单（仅列表内地址通过）和可选的黑名单。
- **速率限制**：按客户端地址的滑动窗口，默认每 60 秒 300 次请求。
- **封禁列表**：多次触发攻击模式匹配的地址会被临时封禁。
- **攻击模式扫描**：扫描 URL 路径、查询字符串和请求头（XSS、SQL 注入、路径遍历等）。命中返回 `400`。`ssrf` 类别已关闭，因为浏览器会把页面地址作为 `Referer` 发送，而 guard-core 4.4.0 会将其中的 localhost 和局域网地址标记出来，因此不筛查 SSRF 模式；待引擎修复发布后再重新评估。

被拒绝的请求返回 `429`（速率限制）或 `403`（IP 列表、封禁）。防护位于 `CORSMiddleware` 内，因此这些响应仍带 CORS 头，浏览器显示真实状态而非不透明的 CORS 错误。

## 不做的功能

- 从不读取请求体。提示词、文档和聊天消息是自由文本，因此关闭请求体扫描（`detection_scan_body=False`）。
- 不筛查 WebSocket 握手（`/ws`），只筛查 HTTP 请求。
- 不使用 Redis。计数器位于各后端进程内，因此当 `backend_workers` 大于 1 时，实际限制最多为 `rate_limit` 乘以 worker 数。
- 不添加安全头、不处理 CORS、不发送遥测（`security_headers`、`enable_cors` 和 `enable_agent` 均关闭）。

截至 fastapi-guard 8.1.1 / guard-core 4.4.0，有两点引擎行为值得注意（后续引擎版本可能改变）：引擎默认 `exclude_paths`（`/docs`、`/redoc`、`/openapi.json`、`/openapi.yaml`、`/favicon.ico`、`/static`）中的路径仅跳过攻击模式扫描，速率限制和 IP 列表仍然适用；白名单内的地址同时跳过攻击模式扫描和速率限制。

## 启用

1. 在拥有服务器的环境中安装可选的防护依赖：

   ```bash
   # 源码检出：
   python scripts/install_extras.py guard
   # 已安装服务器的 wheel 环境：
   python -m pip install "deeptutor[guard]"
   ```

   在 Docker 中，像其他 extras 一样在服务上设置 `DEEPTUTOR_EXTRAS: "guard"`（参见 [CONTAINERIZATION.md](CONTAINERIZATION.md)）。

2. 在 `data/user/settings/system.json` 中启用：

   ```json
   {
     "security_guard": {
       "enabled": true,
       "rate_limit": 300,
       "rate_limit_window": 60,
       "trusted_proxies": ["127.0.0.1", "::1"],
       "exempt_ips": ["127.0.0.1", "::1"],
       "whitelist": [],
       "blacklist": []
     }
   }
   ```

   在 Docker 外，同样的键可通过进程环境变量覆盖：`DEEPTUTOR_GUARD_ENABLED`、`DEEPTUTOR_GUARD_RATE_LIMIT`、`DEEPTUTOR_GUARD_RATE_LIMIT_WINDOW`、`DEEPTUTOR_GUARD_TRUSTED_PROXIES`、`DEEPTUTOR_GUARD_EXEMPT_IPS`、`DEEPTUTOR_GUARD_WHITELIST` 和 `DEEPTUTOR_GUARD_BLACKLIST`（逗号分隔的 IP 或 CIDR 范围）。Docker 入口点忽略进程环境变量覆盖，因此容器内请使用 `system.json`。

3. 重启后端。

若 `enabled` 为 true 但未安装 `fastapi-guard`，后端拒绝启动并打印安装方法。无效的 IP 或 CIDR 条目也会导致启动失败。

### 设定速率限制

该限制统计来自同一客户端地址的**每个** HTTP 请求，包括前端的后台轮询（多个面板每 1 到 5 秒轮询一次）。每 60 秒 300 次可为几个打开的面板留出余量；而 fastapi-guard 8.1.1 在未配置时使用的每 60 秒 10 次会在正常使用中触发。Next.js 前端自行提供静态资源，因此这些请求不会到达后端，也不计入。

## 客户端地址与代理

防护通过请求的 socket 对端识别客户端。仅当对端列在 `trusted_proxies`（默认：仅回环）中时才信任 `X-Forwarded-For`。来自其他对端的请求其 `X-Forwarded-For` 会被忽略。

Uvicorn 也可在应用看到请求前根据 `X-Forwarded-For`（以及 `FORWARDED_ALLOW_IPS`）重写对端，这会让客户端自行选择身份。所有 DeepTutor 启动路径都已传入 `proxy_headers=False` / `--no-proxy-headers`（`deeptutor/api/run_server.py`、`deeptutor_cli/main.py`、`deeptutor/runtime/launcher.py`、两个 `Dockerfile` 启动行），因此防护看到真实对端。`tests/api/test_security_guard.py` 通过 `uvicorn.Config(...).load()` 检查这一点。

若自行启动后端，请保留该标志：

```bash
python -m uvicorn deeptutor.api.main:app --host 0.0.0.0 --port 8001 --no-proxy-headers
```

若后端前有反向代理，将其地址加入 `trusted_proxies`；否则所有客户端看起来都是代理。因此封禁私有地址时防护会发出警告。

双栈绑定：在 guard-core 4.4.0 中，以 IPv4 映射形式（`::ffff:a.b.c.d`，如双栈 `::` 绑定时）报告的对端不会与 IPv4 的 `trusted_proxies` 条目匹配，因此反向代理的 `X-Forwarded-For` 被忽略，其所有客户端共享代理的桶。默认启动路径绑定 `0.0.0.0`，不受影响。若绑定 `::`，请同时列出每个代理的映射形式，例如 `["10.0.0.0/8", "::ffff:10.0.0.0/104"]`（已用 guard-core 4.4.0 验证，并有测试覆盖）。后续 guard-core 版本可能不再需要。

**内置前端。** Next.js 代理（`web/proxy.ts`）会从浏览器请求中删除 `X-Forwarded-For` 及其他转发头（`web/lib/backend-forward.ts`），且不添加浏览器地址。通过它到达后端的请求都来自前端自身地址（默认布局中为回环），不带转发头。单一共享限制会同时限流所有用户，因此 `exempt_ips`（默认 `127.0.0.1` 和 `::1`）使该地址免于速率限制。因此通过前端时，请求会接受攻击模式和黑名单筛查，但不限速。按客户端限制适用于直接到达后端的请求，或通过设置了 `X-Forwarded-For` 且列在 `trusted_proxies` 中的反向代理到达的请求（当 `exempt_ips` 包含回环时，携带 `X-Forwarded-For` 的回环请求按转发的客户端限速）。`exempt_ips` 仅跳过速率限制：黑名单、封禁和攻击模式扫描仍适用于列表内地址。防护不会封禁回环地址。

**白名单与内置前端。** 若设置了 `whitelist`，前端流量（回环）会被拒绝，除非回环在列表中。使用内置前端时，请将 `127.0.0.1` 和 `::1` 加入 `whitelist`。此时白名单和黑名单仅管控直接后端访问，因为前端隐藏了浏览器地址。

## 实现与验证

- [deeptutor/api/security_guard.py](../deeptutor/api/security_guard.py)
- [tests/api/test_security_guard.py](../tests/api/test_security_guard.py)