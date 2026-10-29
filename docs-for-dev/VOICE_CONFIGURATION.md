# 语音配置与适配器开发

[手册](README.md) · [Agent 设置](AGENT_SETUP.md) · [视频学习](watching-workspace.md)

TTS（语音合成）和 STT（语音识别）在语音设置中有各自的提供商配置和默认值，独立于聊天模型。

## 配置与验证

1. 选择一个已实现的提供商，提供其端点、准确的模型/资源 ID 和凭据类型。
2. 在提供商支持时通过发现机制获取音色。音色 ID 不是永久内置目录；账户专属音色和模型 ID 仍可编辑。列表中出现的音色不代表账户已获授权。
3. 选择该模型系列支持的语言、格式和其他选项。
4. 预览一段短样本，然后保存/选择默认值。预览使用当前表单内容，不改变已保存的默认值。
5. 保存后验证实际的朗读或转写效果。模型列表请求成功不能证明语音合成或识别可用。

预览会调用提供商，可能产生用量。表单变更会使旧预览失效；进行中的预览可以取消。请求超时接受 5–600 秒，默认 60 秒。短样本有助于定位配置错误。

## 提供商边界

- **火山引擎语音：** 语音凭据与 Ark LLM 密钥不同。适配器接受 Speech API Key 认证和较旧的 App ID/access-token 形式。TTS 资源和音色系列需匹配。STT 使用录音文件 API；音频归一化可能需要系统 FFmpeg。
- **阿里云 DashScope：** Qwen3 TTS 和 Qwen-Audio TTS 有各自的原生端点和参数。适配器会归一化支持的官方兼容模式 base；私有端点必须实现所选协议。密钥和端点区域需匹配。通过发现机制和实际预览确认音色，不要假设旧示例仍然可用。
- **其他适配器：** 兼容的聊天端点不代表兼容的语音 API。模型选项和实时发现决定支持的音色和格式。

保留认证、区域/模型、权限、速率限制、超时和音频下载等各类错误的区分，不暴露含凭据的响应。保留取消和已配置的超时，不要添加无界重试。

## 源码映射

| 职责 | 实现 |
| --- | --- |
| 配置与解析 | [voice/config.py](../deeptutor/services/voice/config.py) |
| 模型/参数提示 | [voice/options.py](../deeptutor/services/voice/options.py) |
| 音色发现 | [voice/discovery.py](../deeptutor/services/voice/discovery.py) |
| 预览 | [voice/preview.py](../deeptutor/services/voice/preview.py) |
| 提供商协议 | [voice/adapters/](../deeptutor/services/voice/adapters/) |
| 音频转换 | [voice/audio.py](../deeptutor/services/voice/audio.py) |

添加适配器时，需测试凭据处理、端点归一化、解码、超时/取消和错误脱敏。实时检查与模拟协议测试需分开报告。提供商目录、可用性和计费不在本仓库范围内。