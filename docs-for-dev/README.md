# DeepTutor 开发者与 Agent 手册

从这里开始安装、诊断或修改 DeepTutor。本手册描述 **1.7.0 源码树**；处理其他发布版本时，请使用同一标签下的文档。版本以 [`deeptutor/__version__.py`](../deeptutor/__version__.py) 为准。除非页面另有说明，以下命令均在仓库根目录执行。

## 选择阅读路径

| 你的任务 | 按顺序阅读 | 完成证据 |
| --- | --- | --- |
| 安装并配置应用 | [Agent 安装](AGENT_SETUP.md) → [存储](DATA_STORAGE.md) | 依赖就绪、模块就绪、实际模型响应、应用可访问 |
| 修复现有安装 | [安装故障排查](AGENT_SETUP.md#6-troubleshooting) → 下方相关主题 | 原先失败的操作在同一实例和工作区中恢复正常 |
| 开发变更 | [开发](DEVELOPMENT.md) → [架构](ARCHITECTURE.md) → [测试](TESTING.md) | 针对性回归测试及适用的项目检查 |
| 添加工具或能力 | [工具与能力契约](TOOLS_AND_CAPABILITIES.md) → [模块](MODULES.md) | 注册、可用性、执行及终态结果测试 |
| 构建第三方扩展 | [插件](PLUGINS.md) → [可运行的 Learning Echo 示例](examples/learning_echo/README.md) | Wheel、清单、审批、worker 及前端检查 |
| 排查流式传输或记忆 | [事件与进度](EVENTS_AND_PROGRESS.md) → [运行时设计](RUNTIME_DESIGN.md) | 持久化/重放及取消证据 |
| 部署或升级 | [模块](MODULES.md) → [容器化](CONTAINERIZATION.md) → [存储](DATA_STORAGE.md) | 版本一致、数据保留、就绪状态及一次代表性对话 |

对于具备终端能力的 Agent，编辑前请先阅读本索引、[AGENTS.md](../AGENTS.md) 及任务对应指南。[SKILL.md](../SKILL.md) 是精简的 CLI 参考。执行示例前，先检查检出状态、工作区变更、Python 环境及运行时主目录。将现有用户数据和凭据与一次性测试夹具分开。决定重装前，先诊断失败的安装。

## 主题地图

### 运行时与扩展开发

- [架构与源码归属](ARCHITECTURE.md)：入口点、持久对话、前端边界及配置。
- [工具与能力](TOOLS_AND_CAPABILITIES.md)：单次工具、拥有对话的流水线、上下文门控及结果封装。
- [模块](MODULES.md)：分发归属、安装及生命周期。
- [插件](PLUGINS.md)：清单、审核、worker、HTTP 及静态页面。
- [阅读扩展](READING_EXTENSIONS.md)：经验证的选择上下文及有界结果。
- [事件与进度](EVENTS_AND_PROGRESS.md)：对话重放与索引进度及进程内通知的区别。
- [运行时设计](RUNTIME_DESIGN.md)：懒加载、隔离解析、私有模型历史及提示缓存可观测性。
- [远程 Hermes 后端](remote-hermes-backend.md)：网关协议及 Connected Agents 集成。

### 存储、部署与功能契约

- [数据存储](DATA_STORAGE.md) 与 [工作区](workspaces.md)：运行时主目录、账户/工作区范围、备份、资源分配及迁移。
- [容器化](CONTAINERIZATION.md) 与 [请求筛查](SECURITY_GUARD.md)：部署配置、OAuth 回调端口及代理边界。
- [知识迁移](KNOWLEDGE_MIGRATION.md)、[索引恢复](INDEXING_RECOVERY.md) 及 [源可视化](SOURCE_VISUALS.md)：摄取、发布、原始证据及可视化评估。
- [科学研究](SCIENTIFIC_RESEARCH.md)：论文工具、证据及 MCP 授权。
- [练习](PRACTICE.md)：题目导入、复习记录及调度。
- [视频学习](watching-workspace.md) 与 [语音](VOICE_CONFIGURATION.md)：阅读媒体迁移、字幕、语音适配器及验证。
- [答案质量诊断](REASONING_SAFETY_CHECKLIST.md)：可复现的检索、引用、上下文及评估失败。

## 保持本手册的实用性

每个契约保留一个权威页面并链接到它。每个主题应指向其实现归属或回归测试。当命令、路由、归属或存储发生变化时，更新受影响的示例；生成的 Schema 和模块清单优先于文字描述。记录检查所证明的内容，包括提供商/网络或平台限制。

任务计划、一次性测试计数、截图、私有路径、下载的参考仓库及无关的产品研究不要放入本手册。执行证据请使用 PR 或本地被忽略的输出。设计提案在实现可定位并验证之前，不代表当前行为。