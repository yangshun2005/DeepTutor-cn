# 本地数据持久化

[手册](README.md) · [工作区](workspaces.md) · [容器](CONTAINERIZATION.md)

DeepTutor 默认安装不需要 PostgreSQL、MySQL 或其他数据库服务器。持久状态写入单个运行时主目录之下，因此一次部署只需挂载、检查和备份一棵目录树。部分状态存储在 SQLite 中，而设置、生成的构建产物、知识库索引、缓存和导入的源文件仍为普通文件。

## 数据存放位置

运行时主目录是启动 DeepTutor 时所在的目录。设置 `DEEPTUTOR_HOME`，或通过 `deeptutor start --home` / `deeptutor init --home` 指定其他目录。在发布的容器中，同一目录树挂载于 `/app/data`。

默认管理员作用域使用以下布局：

| 路径 | 内容 |
| --- | --- |
| `data/user/settings/` | JSON/YAML 设置和提供商凭据 |
| `data/user/chat_history.db` | SQLite 会话、消息、轮次、题库条目和练习数据 |
| `data/user/.runtime/` | 私有运行时状态，如 `workspaces.sqlite3` |
| `data/user/workspace/` | 默认内容工作区及生成的构建产物 |
| `data/user/workspace/notebook/` | 收藏集及其索引（JSON 文件，沿用旧目录名） |
| `data/user/workspace/co-writer/documents/doc_<id>/manifest.json` | 编写的笔记页面、块、修订和页面元数据 |
| `data/user/logs/` | 应用日志 |
| `data/knowledge_bases/` | 知识库索引和带版本的源数据 |
| `data/parse_cache/` | 共享文档解析缓存 |
| `data/users/<user-id>/` | 启用本地多用户认证时，各用户的工作区根目录 |
| `data/partners/<partner-id>/` | 合成 Partner 工作区 |
| `data/system/` | 账户、授权、审计记录、索引和所有者凭据 |

选定的内容工作区可以位于 `data/user` 之外。此时其私有学习状态位于 `<workspace>/.deeptutor/data/`，而账户设置、凭据、Memory 和工作区注册信息仍保留在运行时主目录中。运行 `deeptutor workspace show` 可打印所选工作区的路径。

收藏和编写的笔记是独立的存储。旧版 `notebook_entries` SQLite 表存放题库条目，并非编写笔记块或收藏集的存储。在选定的外部工作区中，这些基于文件的存储位于其私有的 `.deeptutor/data/workspace/` 目录树下，使用相同的 `notebook/` 和 `co-writer/documents/` 子目录。备份整个工作区（包括隐藏的 `.deeptutor` 文件），以同时保留学习记录和笔记/收藏内容。

## 数据库支持

SQLite 是受支持的本地后端。没有受支持的 PostgreSQL 或 MySQL 连接设置；设置 `DATABASE_URL` 或类似环境变量不会重定向 DeepTutor 的存储。

PocketBase 是通过 `data/user/settings/integrations.json` 配置的可选集成。它将受支持的会话集合存储在 PocketBase 中，但它不是 PostgreSQL 适配器，也不会将所有运行时状态迁入 PocketBase。设置、凭据、知识库索引、解析缓存、日志和工作区构建产物仍保留在运行时目录树中。当前 PocketBase 模式为单用户；如需受支持的本地多用户 JSON/SQLite 布局，请将 `integrations.pocketbase_url` 留空。

## 检查与备份状态

在 DeepTutor 创建数据后，从运行时主目录列出私有运行时目录树中的 SQLite 数据库：

```bash
find data -type f \( -name '*.db' -o -name '*.sqlite3' \) -print
```

对于选定的外部内容工作区，检查其私有数据库文件：

```bash
workspace="$(deeptutor workspace show | sed -n 's/^Path: //p')"
if [ -d "$workspace/.deeptutor/data" ]; then
  find "$workspace/.deeptutor/data" -type f \
    \( -name '*.db' -o -name '*.sqlite3' \) -print
fi
```

复制 `data` 或工作区之前，请停止 DeepTutor 及所有 Partner 进程。SQLite 可能有活跃的预写日志文件，因此应用运行时仅复制 `.db` 文件可能丢失近期事务。备份完整的运行时主目录 `data` 目录，以及每个单独配置的内容工作区。每次副本都应视为敏感数据，因为它可能包含凭据、私密对话文本和上传的学习材料。

## 实现与验证

- [deeptutor/runtime/home.py](../deeptutor/runtime/home.py)
- [deeptutor/services/path_service.py](../deeptutor/services/path_service.py)
- [deeptutor/services/workspace/context.py](../deeptutor/services/workspace/context.py)