

> **[中文化] DeepTutor**
>
> 此项目是 [HKUDS/DeepTutor](https://github.com/HKUDS/DeepTutor) 的中文翻译版本。
> - 原项目 Stars: 48925
> - 主语言: Python
> - 许可证: Apache-2.0
> - 翻译日期: 2026-10-29
> - 原始 README: [README_en.md](README_en.md)
>
> 说明：下方 README 正文为**原项目文档的中文翻译**，力求忠于原文；本页顶部的「中文快速上手」为**面向中文用户的补充导读**（非原文内容），目的是让你不用读完整篇 README 就能先把项目跑起来。
>
> 如有翻译不准确之处，欢迎提 Issue 或 PR。

---


## 🚀 中文快速上手（补充导读，非原文）

### 这是什么 / 解决什么问题

DeepTutor 是一个「agent-native 智能学习伴侣」，项目自述为带多智能体协作与 RAG 的个性化辅导系统（pyproject.toml 中的 description）。它面向需要长期、个性化学习辅导的场景，把文档解析、知识库检索、练习与评估、语音/视频阅读等能力整合进一个学习工作区。项目主语言为 Python，许可证 Apache-2.0。

### 快速上手（最短路径）

1. 确认 Python 版本在 3.11 及以上（pyproject.toml 要求 `>=3.11,<3.15`）。
2. 获取代码后，按 README 正文的 Get Started 章节操作——**原文未提供明确的 pip 安装命令或启动命令**，请参考下方 README 正文。
3. 若使用 Docker 方式，Dockerfile 中给出的构建与运行命令为：
   ```
   docker build -t deeptutor:local .
   docker run -p 127.0.0.1:3782:3782 -p 127.0.0.1:8001:8001 \
     -v deeptutor-data:/app/data deeptutor:local
   ```
4. 首次启动时，运行配置会在 `data/user/settings` 下生成。
5. 在 Web 的 Settings 页面（或通过 `model_catalog.json`）配置模型 provider。

### 环境要求

- **操作系统**：原文未明确限定；Dockerfile 基于 `node:22-slim` 构建前端，未说明宿主系统限制。
- **Python**：`>=3.11,<3.15`（pyproject.toml）。
- **前端**：Next.js 16（README 徽章）；Docker 构建阶段使用 Node 22。
- **依赖**：核心依赖包括 openai、FAISS（注释中提及为编译型基础依赖）、mcp、pydantic 等，完整清单见 `requirements/full.txt`。
- **Docker**：可选，Dockerfile 暴露 3782 与 8001 两个端口。
- 国内网络环境下安装 npm/pip 依赖可能需要更换镜像源或加速下载（原文未说明具体镜像配置）。

### 常见坑与注意事项

- **需要配置模型 provider**：首次启动后必须配置 provider 档案，否则无法正常使用；原文未说明是否必须自备 API Key，但依赖中包含 `openai` 包。
- **编译型依赖**：注释提到 FAISS 等编译型基础依赖需有兼容 wheel，Python 3.14 的支持依赖这些 wheel 的发布情况。
- **Docker 构建耗时**：前端构建阶段设置了较长的 npm 超时与重试（fetch-timeout 600000、retries 5），构建可能较慢。
- **数据持久化**：Docker 运行示例挂载了 `deeptutor-data` 卷到 `/app/data`，不挂载可能导致配置与数据丢失。

---



---

<div align="center">

<p align="center"><img src="assets/figs/logo/color-full.png" alt="DeepTutor" width="480"></p>
<h1 align="center">DeepTutor：终身个性化辅导</h1>

<p align="center">
  <a href="https://deeptutor.info" target="_blank"><img alt="文档 — deeptutor.info" src="https://img.shields.io/badge/Docs-deeptutor.info%20%E2%86%97-0A0A0A?style=for-the-badge&labelColor=F5F5F4" height="36"></a>&nbsp;
  <a href="https://deeptutor.info/collaborate/" target="_blank"><img alt="合作 — 与我们共事" src="https://img.shields.io/badge/Collaborate-work%20with%20us%20%E2%86%97-0A0A0A?style=for-the-badge&labelColor=F5F5F4" height="36"></a>
</p>

<p align="center">
<a href="https://www.star-history.com/hkuds/deeptutor"><picture><source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/badge?repo=HKUDS/DeepTutor&amp;type=trending&amp;theme=dark" /><source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/badge?repo=HKUDS/DeepTutor&amp;type=trending" /><img alt="GitHub 当日热门仓库" src="https://api.star-history.com/badge?repo=HKUDS/DeepTutor&amp;type=trending" height="55" /></picture></a>&#32;<a href="https://trendshift.io/repositories/17099"><img src="https://trendshift.io/api/badge/trendshift/repositories/17099/daily" alt="HKUDS/DeepTutor | Trendshift" width="250" height="55"></a>&#32;<a href="https://www.star-history.com/hkuds/deeptutor"><picture><source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/badge?repo=HKUDS/DeepTutor&amp;theme=dark"><source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/badge?repo=HKUDS/DeepTutor"><img alt="Star 历史排名" src="https://api.star-history.com/badge?repo=HKUDS/DeepTutor" width="185" height="55"></picture></a>
</p>

<p align="center">
  <a href="README.md"><img alt="English" height="40" src="https://img.shields.io/badge/English-BCDCF7"></a>&nbsp;
  <a href="assets/README/README_CN.md"><img alt="简体中文" height="40" src="https://img.shields.io/badge/简体中文-CDCFD4"></a>&nbsp;
  <a href="assets/README/README_TW.md"><img alt="繁體中文" height="40" src="https://img.shields.io/badge/繁體中文-CDCFD4"></a>&nbsp;
  <a href="assets/README/README_JA.md"><img alt="日本語" height="40" src="https://img.shields.io/badge/日本語-CDCFD4"></a>&nbsp;
  <a href="assets/README/README_ES.md"><img alt="Español" height="40" src="https://img.shields.io/badge/Español-CDCFD4"></a>&nbsp;
  <a href="assets/README/README_FR.md"><img alt="Français" height="40" src="https://img.shields.io/badge/Français-CDCFD4"></a>&nbsp;
  <a href="assets/README/README_AR.md"><img alt="Arabic" height="40" src="https://img.shields.io/badge/Arabic-CDCFD4"></a>&nbsp;
  <a href="assets/README/README_RU.md"><img alt="Русский" height="40" src="https://img.shields.io/badge/Русский-CDCFD4"></a>&nbsp;
  <a href="assets/README/README_HI.md"><img alt="Hindi" height="40" src="https://img.shields.io/badge/Hindi-CDCFD4"></a>&nbsp;
  <a href="assets/README/README_PT.md"><img alt="Português" height="40" src="https://img.shields.io/badge/Português-CDCFD4"></a>&nbsp;
  <a href="assets/README/README_TH.md"><img alt="Thai" height="40" src="https://img.shields.io/badge/Thai-CDCFD4"></a>&nbsp;
  <a href="assets/README/README_PL.md"><img alt="Polski" height="40" src="https://img.shields.io/badge/Polski-CDCFD4"></a>
</p>

[![Python 3.11+](https://img.shields.io/badge/Python-3.11%2B-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/downloads/)
[![Next.js 16](https://img.shields.io/badge/Next.js-16-000000?style=flat-square&logo=next.js&logoColor=white)](https://nextjs.org/)
[![License](https://img.shields.io/badge/License-Apache_2.0-blue?style=flat-square)](LICENSE)
[![GitHub release](https://img.shields.io/github/v/release/HKUDS/DeepTutor?style=flat-square&color=brightgreen)](https://github.com/HKUDS/DeepTutor/releases)
[![arXiv](https://img.shields.io/badge/arXiv-2604.26962-b31b1b?style=flat-square&logo=arxiv&logoColor=white)](https://arxiv.org/abs/2604.26962)

[![Discord](https://img.shields.io/badge/Discord-Community-5865F2?style=flat-square&logo=discord&logoColor=white)](https://discord.gg/eRsjPgMU4t)
[![Feishu](https://img.shields.io/badge/Feishu-Group-00D4AA?style=flat-square&logo=feishu&logoColor=white)](./Communication.md)
[![WeChat](https://img.shields.io/badge/WeChat-Group-07C160?style=flat-square&logo=wechat&logoColor=white)](https://github.com/HKUDS/DeepTutor/issues/78)

[功能特性](#-核心特性) · [快速开始](#-快速开始) · [探索 DeepTutor](#-探索-deeptutor) · [CLI](#%EF%B8%8F-deeptutor-cli--agent-原生接口) · [生态](#-生态--eduhub--技能社区) · [社区](#-社区)

</div>

---

> 🤝 **欢迎任何形式的贡献！** 在 [`Roadmap`](https://github.com/HKUDS/DeepTutor/issues/498) 为路线图条目投票或提出新条目，并查看我们的[贡献指南](CONTRIBUTING.md)，了解分支策略、编码规范以及如何开始。

### 📦 发布版本

> **[2026.10.28]** [v1.7.0](https://github.com/HKUDS/DeepTutor/releases/tag/v1.7.0) — 模块化安装、焕新的学习工作区、Dito 块笔记本、带旁白的视频课程、可恢复的嵌入检查点，以及更可靠的辅导。

<details>
<summary><b>历史版本（超过 1 周前）</b></summary>

> **[2026.10.19]** [v1.6.15](https://github.com/HKUDS/DeepTutor/releases/tag/v1.6.15) — 学习路径变更审核、分组测评、有界辅导记忆、可靠的聊天恢复、更稳健的解析、检索对比，以及原生 Gemini 语音。

> **[2026.10.8]** [v1.6.14](https://github.com/HKUDS/DeepTutor/releases/tag/v1.6.14) — 原始来源可视化练习、可恢复的索引、统一的视频阅读、科研工具、受管插件、学习日志，以及提供商设置改进。

> **[2026.10.4]** [v1.6.13](https://github.com/HKUDS/DeepTutor/releases/tag/v1.6.13) — 阅读与听力改进、题库练习、弹性文档解析、可配置语音、工作区可见附件、波兰语，以及新提供商。

> **[2026.9.27]** [v1.6.12](https://github.com/HKUDS/DeepTutor/releases/tag/v1.6.12) — 工作区知识库迁移、Kiwix 归档、来源图表、任务看板、德语界面，以及聊天恢复。

> **[2026.9.24]** [v1.6.11](https://github.com/HKUDS/DeepTutor/releases/tag/v1.6.11) — 法语和乌克兰语界面、带聊天内选择操作的阅读文件夹、发送给视觉模型的图表、分页 Office 预览，以及自同步知识库。

> **[2026.9.22]** [v1.6.10](https://github.com/HKUDS/DeepTutor/releases/tag/v1.6.10) — 原生 LightRAG 角色模型、记录并强制构建来源的已发布索引、跟随解析引擎的 PDF 附件、可见的截断，以及未过滤的提供商选择。

> **[2026.9.21]** [v1.6.9](https://github.com/HKUDS/DeepTutor/releases/tag/v1.6.9) — 基于文件夹的学习工作区、每日练习、重新设计的设置、更清晰的流式对话、持久用量统计，以及可恢复归档与显式永久删除。

> **[2026.9.14]** [v1.6.8](https://github.com/HKUDS/DeepTutor/releases/tag/v1.6.8) — 已删除聊天的回收站、跨全部对话历史的搜索、可越过知识库的工具，以及一批静默失败的修复。

> **[2026.9.11]** [v1.6.7](https://github.com/HKUDS/DeepTutor/releases/tag/v1.6.7) — 修复版本：整本书变成空章节、测验无输出、模型草稿混入正文、公式原样打印，以及无法提交的卡片。

> **[2026.9.8]** [v1.6.6](https://github.com/HKUDS/DeepTutor/releases/tag/v1.6.6) — 修复版本：无法提交的答案、说谎的复制按钮、合作伙伴连接的知识库、Docker 内的 Codex 登录，以及首页路由减重 100 KB。

> **[2026.9.6]** [v1.6.5](https://github.com/HKUDS/DeepTutor/releases/tag/v1.6.5) — 可指向任意文件夹的内容工作区、适用于所有语言的单一 `exec` 工具、可门控其工具的 Mastery Path 模式，以及评估就绪状态的设置。

> **[2026.9.3]** [v1.6.4](https://github.com/HKUDS/DeepTutor/releases/tag/v1.6.4) — 更快的隔离运行时、可控的 Book 生成、来源完整的 Mastery 路径与 Chat 交接、持久阅读、统一活动界面、可恢复会话，以及显式的逐模型 API 能力。

> **[2026.9.2]** [v1.6.3](https://github.com/HKUDS/DeepTutor/releases/tag/v1.6.3) — 破坏性前后端重构、严格的规范路由与可恢复流，以及学习者/监护人账户、有据阅读、WeKnora、更广的解析、Python 3.14 和 DashScope 媒体。

> **[2026.8.31]** [v1.6.2](https://github.com/HKUDS/DeepTutor/releases/tag/v1.6.2) — 沉浸式 YouTube 学习、插件驱动的 Visualize 目录、三种新 agent harness、更安全的阅读引用、多格式 MinerU、实时 Partner 频道状态，以及引导式更新。

> **[2026.8.30]** [v1.6.1](https://github.com/HKUDS/DeepTutor/releases/tag/v1.6.1) — 一个厂商密钥关联其服务的所有服务、用于后台任务的任务模型、可搜索导航的设置、可自行排列的侧边栏，以及第一方 **LightRAG**。

> **[2026.8.27]** [v1.6.0](https://github.com/HKUDS/DeepTutor/releases/tag/v1.6.0) — 忠实还原的 EPUB 阅读与标注、带 Little Tutor 和 Ask Questions 的课程、有界网页来源同步、共享 Books 与私有学习状态，以及 Serply/原生搜索。

> **[2026.8.25]** [v1.5.17](https://github.com/HKUDS/DeepTutor/releases/tag/v1.5.17) — 每位成员拥有带私密对话和可关联聊天账户的 Partners、作为知识来源的 GitHub 仓库、**Antigravity CLI**、浏览器微信扫码登录，以及 `deeptutor doctor`。

> **[2026.8.22]** [v1.5.16](https://github.com/HKUDS/DeepTutor/releases/tag/v1.5.16) — 可连接的 **MarginNote 4** 库及其插件填充、可再次翻动的 Book 页面，以及不再在网关后失效的工具调用 ID、嵌入和温度限制。

> **[2026.8.20]** [v1.5.15](https://github.com/HKUDS/DeepTutor/releases/tag/v1.5.15) — 自托管的 **PageIndex OSS** 与推理检索、终于可归档的题库、第三方工具/能力插件，以及 **Apache Tika** 解析。

> **[2026.8.19]** [v1.5.14](https://github.com/HKUDS/DeepTutor/releases/tag/v1.5.14) — **沉浸式阅读**：文档在对话旁打开，逐页引用；DeepTutor 通过聊天自我配置；可浏览和写入的 IMA 库；笔记本控制台。

> **[2026.8.17]** [v1.5.13](https://github.com/HKUDS/DeepTutor/releases/tag/v1.5.13) — Books 在编译时流式输出、跟踪进度并导出为 Markdown；批准书脊前的成本估算；以及从记忆中提取的首页起始建议。

> **[2026.8.13]** [v1.5.12](https://github.com/HKUDS/DeepTutor/releases/tag/v1.5.12) — 重建的网页搜索，新增六个提供商（**Doubao**、**Bocha**、**Zhipu**、**Firecrawl**、**Qianfan**、**Aliyun IQS**）、**LiteParse** 解析引擎、凭证变更时重连的 MCP 服务器，以及 **CodeBuddy** + **OrcaRouter**。

> **[2026.8.10]** [v1.5.11](https://github.com/HKUDS/DeepTutor/releases/tag/v1.5.11) — DSML 工具调用周围的文本不再消失、截断的回复继续而非结束、设置中显示实时内存用量，以及 LightRAG 索引移出事件循环。

> **[2026.8.7]** [v1.5.10](https://github.com/HKUDS/DeepTutor/releases/tag/v1.5.10) — 每个账户登录自己的 **Codex**、模型输出语言成为独立设置、空工具调用被拒绝而非重试，以及上传不再阻塞循环。

> **[2026.8.4]** [v1.5.9](https://github.com/HKUDS/DeepTutor/releases/tag/v1.5.9) — 原生端点上的 Gemini **Embedding 2**、逐模型 **reasoning effort** 控制、**Novita AI** 网关、查询的检索角色，以及保留全部 `data/` 的 Compose 部署。

> **[2026.8.2]** [v1.5.8](https://github.com/HKUDS/DeepTutor/releases/tag/v1.5.8) — 内存：开发服务器真正的堆上限、源码安装提供生产构建、有界 LLM 客户端和索引缓存，以及针对偶发 500 的 keep-alive 修复。

> **[2026.7.31]** [v1.5.7](https://github.com/HKUDS/DeepTutor/releases/tag/v1.5.7) — 逐账户的 **MCP Services** 存储、导师可运行的 101 个 **CLI Apps**、移出沙箱可达范围的凭证，以及移动端布局。

> **[2026.7.29]** [v1.5.6](https://github.com/HKUDS/DeepTutor/releases/tag/v1.5.6) — 远程 **Codex** 登录可在 SSH 隧道后完成、生成文件在 Activity 中有独立卡片、非英语语言不再坍缩为中文，以及书籍创建不再超时。

> **[2026.7.26]** [v1.5.5](https://github.com/HKUDS/DeepTutor/releases/tag/v1.5.5) — 通过 **OpenAI Codex** OAuth 使用 ChatGPT 套餐登录、**Eden AI** 提供商、报告所持内容的知识库、可追溯的 `rag` 引用，以及无需变通方法的 GraphRAG 索引。

> **[2026.7.24]** [v1.5.4](https://github.com/HKUDS/DeepTutor/releases/tag/v1.5.4) — 维护性清理：回答后"生成中