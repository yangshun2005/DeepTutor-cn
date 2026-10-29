# 知识迁移指南

[手册](README.md) · [相关指南](INDEXING_RECOVERY.md)

本指南介绍如何将 Obsidian、Hermes 或其他基于 Markdown 的工具中的现有笔记迁移到当前知识中心。如需在工作区之间迁移现有 DeepTutor 数据，请使用[工作区数据迁移](workspaces.md)。

## 选择迁移路径

| 路径 | DeepTutor 保留的内容 | 适用场景 | 取舍 |
| --- | --- | --- | --- |
| 连接 Obsidian 仓库 | 指向仓库的实时指针 | 继续以 Obsidian 作为唯一数据源 | 仓库不会复制到向量索引，因此不是 RAG 语料库 |
| 导入已索引的知识库 | 将 Markdown 及其他支持的文件复制到 `raw/`，然后建立索引 | 用所选检索引擎搜索复制后的语料库 | 原始笔记的后续修改不会自动导入 |

DeepTutor 没有专门的一键 Hermes 导入器。请将 Hermes 笔记导出为 Markdown，然后按导入知识库的路径操作。请先在笔记工具中检查导出结果；DeepTutor 不会转换专有格式。

## 准备源数据

1. 在连接或导入之前，对源仓库或导出内容做完整备份。
2. 将笔记导出为 `.md` 或 `.markdown`。如果笔记工具支持，请一并包含关联的图片和文档资源。
3. 如果应用导出的是 HTML 或其他格式，请在 DeepTutor 之外将其转换为 Markdown，抽查样本，并确认链接和附件仍可解析。
4. 从上方表格中选择一条路径。两者可以同时使用：连接实时 Obsidian 仓库用于笔记工作，需要可检索索引时再导入一份副本。

## 连接 Obsidian

当 Obsidian 仍是你的主要编辑器，且希望 DeepTutor 浏览并追加到同一批笔记时，使用此路径。

1. 在 DeepTutor 中打开 **Knowledge Center**，选择 **Create KB**，然后选择 **Link existing** 和 **Obsidian**。
2. 输入 DeepTutor 服务器可访问的仓库文件夹绝对路径，例如 `/Users/example/Documents/MyVault`。不要输入其他机器上的路径。
3. 为知识库指定一个独特的名称并连接。
4. 在 Chat 中选择该知识库。Obsidian 工具随后会操作所选仓库。

DeepTutor 创建的是指针，而非上传或索引。它能识别 Markdown 笔记，跟踪仓库链接和标签，并忽略 `.obsidian`、`.trash` 和 `.git` 内部内容。写入是追加式的：可以创建笔记、追加内容或添加 frontmatter 属性，但助手不会删除或重写现有笔记正文。删除知识库只会移除 DeepTutor 指针；仓库保持不变。

对于自托管容器，请挂载宿主机仓库，使相同路径在服务器容器内可见，然后连接服务器可见的路径。仅在宿主机上有效的路径不够。

如需对 Obsidian 笔记副本进行语义检索，请将该副本作为单独的已索引知识库导入。保留已连接的仓库用于实时编辑，使两种用途保持清晰。

## 导入 Markdown 或 Hermes 笔记

当你需要复制并建立索引的语料库时，使用此路径。源笔记保留在原位置。

### Web

1. 将笔记和资源导出到本地 Markdown 文件夹。
2. 在 DeepTutor 中打开 **Knowledge Center**，选择 **Create KB**。
3. 选择 **Create new** 和一个检索引擎。LlamaIndex 是默认选项；仅当其他引擎的取舍适合该语料库时才选择。
4. 上传文件、文件夹或 `.zip` 压缩包。
5. 等待解析和索引完成，然后确认几篇代表性笔记可检索，再删除任何源备份。

**文件夹上传**会保留相对目录结构。`.zip` 压缩包会以防御性方式展开：嵌套目录被展平，不支持的成员被跳过，隐藏和系统条目被跳过，展平后重名的基名被跳过。当笔记标题在不同文件夹中重复，或文件夹层级有意义时，优先使用文件夹上传。

上传界面会显示支持的扩展名和单文件限制。当前 Markdown 扩展名为 `.md` 和 `.markdown`；其他支持的文本、文档、电子表格、演示文稿、EPUB、图片和源代码格式均按同一上传策略接受。

### CLI

在导出目录中：

```bash
deeptutor kb create hermes-notes --docs-dir /path/to/hermes-export
```

对于单个文件或后续添加：

```bash
deeptutor kb create one-note --doc /path/to/note.md
deeptutor kb add hermes-notes --docs-dir /path/to/more-notes
```

CLI 会从 `--docs-dir` 递归收集支持的文件，然后将每个匹配的文件复制到知识库 `raw/` 目录的顶层。由于重复文件名可能覆盖较早的副本，当文件名重复或层级有意义时，优先使用 Web 文件夹上传。使用 `deeptutor kb list` 和 `deeptutor kb info hermes-notes` 确认创建结果和文档数量。

## 验证迁移

- 打开一篇小型、中型和深层嵌套的样本笔记，检查文本、链接和资源引用是否仍可读。
- 对于已索引的知识库，针对确切的笔记名称、短语和概念运行代表性搜索。确认引用的来源指向预期的笔记。
- 对于 Obsidian，搜索一篇笔记，跟踪一个链接或反向链接，创建一篇临时笔记，然后从 Obsidian 中删除该临时笔记。
- 在确认检索质量和资源引用之前，保留原始备份。

## 实现与验证

- [deeptutor/capabilities/obsidian/vault.py](../deeptutor/capabilities/obsidian/vault.py)
- [deeptutor/api/routers/knowledge.py](../deeptutor/api/routers/knowledge.py)
- [deeptutor_cli/kb.py](../deeptutor_cli/kb.py)