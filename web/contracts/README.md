# 生成的前端契约

后端负责接口契约。不要手动编辑 `schema/` 或 `generated/` 下的文件。

后端契约变更后，在仓库根目录刷新这两层：

```bash
python scripts/export_frontend_contracts.py
cd web && npm run contracts:generate
```

CI 会在后端契约测试中运行 `python scripts/export_frontend_contracts.py --check`，并在前端类型检查前运行 `npm run contracts:check`。漂移失败意味着必须用上述命令重新生成已提交的构建产物。