---
title: CPA反代Codex
category: Blog
tags:
  - GitBlog
published: 2026-05-09T13:05:96+08:00
image: https://image.heavenroad.org/default_cover.webp
slug: slug20260509130596
upload: false
Last Modified: 2026-05-09 15:05:36
---
## CPA 配置
1. 安装 [CPA](https://github.com/router-for-me/CLIProxyAPI)
2. 配置 caddy/nginx
3. CPA 之内 OAuth 中完成 openai 登陆
4. CPA 配置面板中添加 API Key

## Codex 配置
1. 安装 Codex 省略
2. 编辑 `~/.codex/config.toml`

```
model = "gpt-5.5"
model_provider = "vapi" # 和下面的 model_providers.vapi 一致
model_reasoning_effort = "medium"
preferred_auth_method = "apikey"

# 供应商设置
[model_providers.vapi]
name = "CPA"
base_url = "https://cpa.yourdomain.com/v1"
wire_api = "responses" # 使用 /v1/responses 端点
query_params = {}
request_max_retries = 3            # 失败最大重试次数
stream_max_retries = 6            # 流中断后最大重试次数
```

3. 编辑 `~/.codex/auth.json`

```
{
  "OPENAI_API_KEY": "sk-API_KEY_IN_CPA"
}
```

4. 启动 codex 应该就 ok 了
