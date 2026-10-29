# LightRAG 解析器桥接测试夹具

`original.pdf` 由 `generate_original_pdf.py` 生成，是 DeepTutor 的原始测试材料，采用仓库的 Apache-2.0 许可证。它仅包含合成文本、一个公式、一张小表格和一个绘制的矩形，不含任何第三方文档内容。

生成文件 SHA-256：`e2abd3404f3a03d90c36feec19786689892434e730841cbdbd2a1a28875f3a80`。

本目录中的基准解析器输出必须记录确切的源提交或经过认证的 MinerU API 捕获日期、响应 Schema/版本、捕获命令以及 SHA-256。密钥和临时服务元数据绝不能提交。

当前的 MinerU v4 捕获通过以下命令交互式生成：

```bash
.venv/bin/python tests/fixtures/lightrag_bridge/capture_current_mineru.py
```

脚本通过 `getpass` 私密提示输入；不接受在命令参数、环境变量、输出文件或日志中传入令牌。

`mineru-legacy-official-example/content_list.json` 由 `extract_legacy_example.py` 从官方 MinerU 文档提交 `61cc6886fe3edda8aa1c5b8bd2b6eaedddb8af99` 逐字节提取。其来源明确标识为官方 Schema 示例，而非实际解析器运行结果。经过认证的当前捕获仍是 `original.pdf` 的真实输出夹具。

## 冻结映射契约

| MinerU 字段 | LightRAG 公开 Sidecar 字段 | 失败边界 |
| --- | --- | --- |
| `title`、`section_header` 或正数 `text_level` | `IRBlock.heading`、`level`、`parent_headings` | 空标题保留为普通内容，不能创建层级节点。 |
| `text`、`content`、`body`、`code_body` | `IRBlock.content_template` | 未知块若这些非空字段全无，则文档失败。 |
| `rows` 或 `table_body` | `IRTable.rows` 或 `html` | 表格既无行矩阵也无 HTML 时，在分析前失败。 |
| `table_caption`、`header`、`table_footnote` | `caption`、`table_header`、`footnotes` | 格式错误的可选值规范化为空字段；表格主体仍为必填。 |
| `img_path` 或 `path` | 已验证的 `AssetSpec` 加 `IRDrawing.asset_ref` | 绝对路径/路径穿越/缺失/歧义/未冻结的资产导致文档失败。 |
| `image_caption`/`chart_caption`、图像/图表脚注、格式 | `IRDrawing.caption`、`footnotes`、`fmt` | 不允许仅提示或缺失资产的回退。 |
| 公式文本/主体加 `text_format` | `IREquation.latex`、`is_block` | 空公式内容导致文档失败。 |
| `page_idx` 或 `page`，以及 `bbox` | `IRPosition.anchor`、`range` | 整数 `page_idx` 从零基转换为一基；无效 bbox 被省略。 |

原始解析器夹具和预期的 Sidecar 断言均为不可变测试输入。生产代码只能导入 LightRAG 的公开 Sidecar IR 和 writer；上游 MinerU 构建器仅作对比证据，不是运行时依赖。