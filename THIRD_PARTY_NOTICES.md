# 第三方声明

## CSSwitch

- 项目：[SuperJJ007/CSSwitch](https://github.com/SuperJJ007/CSSwitch)
- 源码提交：`4e0af6ba7909dca22f1257b168172ecbe4af4836`
- 许可证：MIT
- 版权：Copyright (c) 2026 shanjunjie
- 借鉴的概念：PKCE 回环登录、认证代次、凭据原子更新、模型目录缓存失效、脱敏操作状态。

DeepTutor 的 Codex OAuth 支持借鉴了上述设计概念，并基于 DeepTutor 自身的设置目录、模型目录和提供商生命周期独立实现。该源码提交的 MIT 许可证文本如下：

```text
MIT License

Copyright (c) 2026 shanjunjie

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## Hermes Agent

- 项目：[NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)
- 源码提交：`7d6db4efb885856078e4d19f804035226df81e0d`
- 许可证：MIT
- 版权：Copyright (c) 2025 Nous Research
- 借鉴的概念：飞书/Lark 设备码机器人注册和 WeCom AI Bot 二维码创建流程，包括其重试和终止错误语义。

DeepTutor 使用自身的异步 HTTP 服务、内存会话模型、合作伙伴配置合并和 Web 管理界面实现这些协议。该源码提交的 MIT 许可证文本如下：

```text
MIT License

Copyright (c) 2025 Nous Research

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## thinking-orbs

- 项目：[Jakubantalik/Libraries](https://github.com/Jakubantalik/Libraries)（`packages/thinking-orbs`）
- 源码提交：`3862ffa345217443b63696a8c331a0664eea4b04`
- 许可证：MIT
- 版权：Copyright (c) 2026 Jakub Antalik
- 内置文件：`web/vendor/thinking-orbs/`

DeepTutor 内置该包的源码，而非依赖已发布的 `thinking-orbs` npm 包，以便点状思考球能采用宿主行的 `currentColor`，而非固定的灰度渐变。这些本地修改列于 `web/vendor/thinking-orbs/index.ts` 顶部，并在每处修改点标注。其余内容均为上游原样，未作修改。该源码提交的 MIT 许可证文本如下：

```text
MIT License

Copyright (c) 2026 Jakub Antalik

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```