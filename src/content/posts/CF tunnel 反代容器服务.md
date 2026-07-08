---
title: CF tunnel 反代容器服务
category: Blog
tags:
  - Cloudflare
  - Tunnel
  - Docker
published: 2026-07-08T09:07:26+08:00
image: https://image.heavenroad.org/default_cover.webp
slug: slug20260708090726
upload: false
Last Modified: 2026-07-08 09:07:86
---
一般服务器上运行了多个服务会用不通的端口，用户请求发到主机的后，WebService进行SSL的解密，并根据域名或路径等规则，分派到各个不同的端口上。这个过程叫反向代理（Reverse Proxy）。

主流采用Nginx/Caddy，我最近一直在用Caddy，因为可以加插件，使用S3保存一份证书，在多个机器上用，不需要所有的机器都自动更新证书。

这半年有几个机器懒得折腾，就尝试了一下Cloudflare的tunnel服务。这个tunnel比开CDN☁️差不多但是我个人感觉更好玩一些。

主要步骤：
1. 在CF面板新建一个Tunnel
2. 在VPS上运行cloudflared（用面板提供的指令）
3. 在CF面板添加一个路由Route到