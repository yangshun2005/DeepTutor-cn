# 科学研究工作流

[手册](README.md) · [相关指南](TOOLS_AND_CAPABILITIES.md)

DeepTutor 挂载了 `preprint`、`research_audit` 和 `research_lit`，分别用于科学论文请求、支持论文搜索的聊天和 Deep Research。它们是受上下文门控的工具，与 [工具与能力](TOOLS_AND_CAPABILITIES.md#mounting-rules) 中描述的八个用户可切换工具相互独立。

- `preprint(paper, action="read", pages=[...])` 读取主 arXiv PDF，保留请求的版本和旧版归档 ID，记录其 SHA-256，并返回页面级文本和原始来源链接。下载上限为 32 MiB，PDF 上限为 500 页；单次读取最多 20 页 / 36,000 字符。覆盖范围和缺失的可提取文本会明确标注。
- `preprint(..., action="figures")` 在调用方未指定页面时，选择图注候选页面。它渲染忠实的原始页面，而非声称自动裁剪图片。对于支持视觉的模型，最多两张图片进入请求本地模型上下文；像素数据永不进入持久化工具元数据。链接标明来源版本和页码。渲染并不证明模型实际检查了每张图片。
- `research_audit(paper, repository, ref="HEAD", files=[...], checks=[...])` 将 GitHub 修订解析为提交，并读取最多八个静态文件。引用检查要求论文引文和代码引文均存在于所获取的来源中。数值差异是语义审查的候选对象：参数映射、单位、默认值和活动路径仍然重要。本地/管理员原生读取可使用现有的 `GITHUB_TOKEN` 环境凭据；其他账户使用各自授权的 MCP 连接，而非借用部署令牌。公共 API 速率限制/访问失败仍视为未验证失败。该工作流不执行下载的仓库代码，也不认证实验复现。方程和方法请使用完整主文本，并报告仍未验证的运行时/环境/数据检查。
- `research_lit(topic, followups=[...])` 搜索主 arXiv 元数据，最多支持三个显式后续查询，并对论文身份去重。其矩阵记录来源、作者、证据级别和覆盖范围。在判定一致/不一致前请阅读全文；Deep Research 的 APPEND 队列支持对矛盾和证据缺口进行递归调查。

报告将主张、方法、数据、指标、结果和局限性，与论文页面和提交文件引用进行对比。作者发现、模型推断、生成的摘要和社区注释保持为不同的证据层。检索不可用属于失败，而非空集或已验证的发现。

## alphaXiv 与自托管研究 MCP

目录包含官方 [alphaXiv MCP 端点](https://www.alphaxiv.org/docs/mcp)，支持 OAuth 和四个研究阅读工具：发现、全文文本、页面查询和仓库文件。此预设不启用库写入工具。需要主证据时，请使用 `get_paper_content` 请求 `fullText=true`；其默认的中间报告是生成的分析。

聊天和 Deep Research 对这些工具使用相同的账户/工作区/合作伙伴授权视图。`preprint(action="mcp", mcp_tool="<qualified name>", arguments={...})` 可调用已公布、已授权的研究阅读操作。自托管的 Feynman 风格 MCP 服务器可通过此绑定暴露审计/文献/arxiv/alphaxiv 操作，或只读的讨论/注释操作。

`preprint(action="discussion")` 读取公开的 alphaXiv 论文页面，并将注释覆盖范围标记为未验证：生成的概览和动态省略的评论不得作为主论文或完整讨论导出呈现。当前官方 MCP 文档未公布专用的评论端点。如果配置的服务器暴露了 `get_comments`、`get_discussions`、`get_annotations` 或 `get_errata`，该输出会明确标记为社区注释层；不可用的工具仍保持不可用。

## 受信任的私有 MCP 源

管理员可通过在管理员 MCP 注册表中设置 `allow_private_network=true` 来批准专用的远程服务器源。编辑器会暴露此设置；禁用的管理员条目可作为批准，而无需公布共享工具连接。账户自己的服务器仅当其确切的 scheme、host 和 port 与当前管理员批准匹配时，才能选择相同的标志。

```json
{
  "servers": {
    "research_origin_approval": {
      "url": "http://research.internal:8080/mcp",
      "allow_private_network": true,
      "enabled": false
    }
  }
}
```

该例外仅允许该源使用 RFC1918 Docker/LAN 地址。自助回环、云元数据/链路本地地址及其他私有源仍被阻止。DNS 和批准在连接/请求边界及工具准入时检查；撤销管理员标志会阻止池化连接的下一次调用。自助重定向保持禁用；共享传输重定向在连接到目标前检查。此设置既不改变 `web_fetch`，也不改变原生论文/仓库下载。旧版管理员拥有的本地/stdio 配置保持原有行为。原生 arXiv 元数据在进程内限速；提供方速率限制错误保持可见。参见 [arXiv API 文档](https://info.arxiv.org/help/api/user-manual.html)。

## 实现与验证

- [deeptutor/tools/research_tools.py](../deeptutor/tools/research_tools.py)
- [deeptutor/services/research/primitives.py](../deeptutor/services/research/primitives.py)
- [tests/services/research/test_primitives.py](../tests/services/research/test_primitives.py)
- [tests/services/research/test_mcp_boundaries.py](../tests/services/research/test_mcp_boundaries.py)