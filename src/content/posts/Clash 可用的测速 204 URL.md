---
title: Clash 可用的测速 204 URL
category: 技术
tags:
  - 翻墙
published: 2026-03-27T08:03:22+08:00
image: https://image.heavenroad.org/Pasted%20image%2020260327082423.png
slug: slug20260327080322
upload: false
Last Modified: 2026-07-15 11:07:50
---

Clash 的 url-test 可以请求一个返回响应 204 的 URL 测试节点的延迟，从而评估节点质量来自动选择最优的线路。

这些 URL 主要是由一些国外的大型互联网公司用于快速检测网络连通性、用户追踪等用途，通过访问一个会返回 204 状态码来判断网络是否畅通，并且通常请求体很小或者为空，以尽量减小网络开销。

需要注意的是，这些 URL 可能会因为服务调整而发生变化，所以在实际使用时还是建议先进行测试和验证。

本文列出多个大公司的 204 的 测试 URL，可以选择更换。

## Clash url-test 示例

```
- name: ♻️ 自动选择
  type: url-test
  url: http://www.gstatic.com/generate_204
  interval: 300
  proxies:
  - 节点一
  - 节点二
  - 节点三
```

## 常见 204 URL

### Google

#### http://www.gstatic.com/generate_204

- Google Chrome 浏览器用来检测网络连通性的 URL，也是 Clash 配置中最常见的

#### http://maps.googleapis.com/maps/api/mapsjs/gen_204

- Google Map

#### http://www.google.com/generate_204

- Google Chrome 浏览器的另一个检测网络连通性的 URL

#### http://www.google-analytics.com/generate_204

- Google Analytics 分析

#### http://connectivitycheck.gstatic.com/generate_204

- Google Chrome 浏览器的另一个检测网络连通性的 URL

#### https://clients3.google.com/generate_204

- 另一个 Google 用于检测网络连通性的 URL

#### http://www.google.com/blank.html

- Google 的一个空白页面，访问会返回 204

#### https://ssl.gstatic.com/ui/v1/icons/mail/images/cleardot.gif

- Gmail 使用的一个空白图片 URL，访问会返回 204

### Cloudflare

#### http://cp.cloudflare.com/generate_204

- CDN 大厂 Cloudflare

### Apple

#### http://captive.apple.com

- 苹果设备用来检测热点门户的 URL，正常访问会返回 204

#### http://www.apple.com/library/test/success.html

- 苹果设备检测网络的另一个 URL

### 微软

#### http://www.msftncsi.com/ncsi.txt

- Microsoft 用来检测 Internet 连接的 URL，访问会返回 204

#### http://www.msftconnecttest.com/connecttest.txt

- Microsoft 另一个用来检测 Internet 连接的 URL，访问会返回 204

#### https://bat.bing.com/action/0

- 微软 Bing 搜索引擎用于追踪用户的 URL，访问会返回 204

### Facebook

#### https://www.facebook.com/common/referer_frame.php

- Facebook 用于追踪用户来源的 URL，访问会返回 204

### Firefox

#### http://detectportal.firefox.com/success.txt

- 火狐浏览器用来检测网络连通性的 URL

### Twitter

#### https://twitter.com/favicon.ico

- Twitter 的 favicon 图标 URL，访问会返回 204

### 其他

#### http://www.v2ex.com/generate_204

#### https://http.cat/204

#### https://httpbin.org/status/204
