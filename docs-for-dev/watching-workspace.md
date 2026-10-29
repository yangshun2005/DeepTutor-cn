# 沉浸式阅读中的视频学习

[手册](README.md) · [相关指南](READING_EXTENSIONS.md)

在 Learning 下打开 **Reading**，将 YouTube 或 Bilibili URL 添加到收藏集，即可在导师旁学习。视频与文档共用同一套阅读材料、转录文本、标注和对话存储。媒体舞台支持转录搜索与跟随、时间戳引用、全屏观看以及当前/下一句字幕。字幕缺失或播放不受支持时会明确提示；导师不得仅凭转录文本声称已观看视频。

新建的时间戳笔记属于 Reading 标注。其位置在编辑器打开时捕获，因此写作时播放进度移动不会带动笔记位置。保存失败会保留草稿以便重试；较早的保存完成不会覆盖较新的文本。

## 已有的 Watching 对话

独立 Watching 已停用。已有的 `/watching/{sessionId}` 和 `/learning/watching/{sessionId}` 链接会打开兼容路由，明确将对话迁移到 Reading。对话 ID 和消息保持不变。旧版视频 JSON 仍作为 provider/缓存记录保留；已有的 **Video Learning** 笔记本记录仍可读取，不会被重写。兼容的转录文本、笔记和保存位置会幂等地复制到 Reading，且已有的 Reading 位置优先于较早导入的位置。

会话列表和检查不会创建材料。迁移在学习者打开旧对话或继续旧轮次时发生。若旧视频缓存缺失，其对话仍可作为普通聊天使用，并记录视频不可用状态；绝不会静默替换为其他视频。其他迁移失败会保留原始偏好并提供重试。

## 配置 Invidious

在管理员设置中打开视频学习设置，输入实例的后端 API 源和公共源，测试连接，并选择 Invidious 作为默认 provider。Reading 的 YouTube 字幕加载器使用该已配置的 provider。Bilibili 保持独立。字幕服务不可用不会使原生可播放的视频无法使用，但缺失的转录文本会可见，导师的证据仍然受限。

`invidious.api_base_url` 必须能被后端访问；`invidious.public_base_url` 必须能被浏览器访问。私有 HTTP 源可在现有已配置实例策略下使用回环、局域网或覆盖网络地址。公共实例要求 HTTPS。设置保存在 `video_learning.json` 中；项目 `.env` 文件会被忽略。

在添加材料对话框中使用 **Browse Invidious**，从账号的订阅和播放列表中搜索或选择。账号令牌保持所有者私有。缺少 `GET:feed`、`GET:playlists` 和 `GET:playlists/*` 的旧令牌需要重新连接；不会请求任何订阅或播放列表变更。DeepTutor 从不索取 Invidious 密码。授权在已配置实例上进行，回调反馈返回 Reading，且不会将令牌放入重定向目标。断开连接失败会保留令牌以便重试。

## 验证与恢复

检查播放、跳转、转录搜索/跟随、时间戳引用以及笔记的保存/编辑/重试。重新打开已迁移的对话，确认其消息历史和来源身份保持不变。同时检查字幕缺失、provider 不可用、窄屏幕和所有者访问边界。在部署或回滚期间保留现有数据目录和所有者身份；不要仅为撤销前端更改而用备份覆盖较新的用户数据。

## 实现与验证

- [deeptutor/reading/](../deeptutor/reading/)
- [web/components/reading/](../web/components/reading/)