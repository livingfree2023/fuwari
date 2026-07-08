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
Last Modified: 2026-07-08 09:07:17
---
一般服务器上运行了多个服务会用不通的端口，用户请求发到主机的后，WebService 进行 SSL 的解密，并根据域名或路径等规则，分派到各个不同的端口上。这个过程叫反向代理（Reverse Proxy）。

主流采用 Nginx/Caddy，我最近一直在用 Caddy，因为可以加插件，使用 S3 保存一份证书，在多个机器上用，不需要所有的机器都自动更新证书。

这半年有几个机器懒得折腾，就尝试了一下 Cloudflare 的 tunnel 服务。这个 tunnel 比开 CDN☁️差不多但是我个人感觉更好玩一些。

主要步骤：
1. 在 CF 面板新建一个 Tunnel
2. 在 VPS 上运行 cloudflared（用面板提供的指令）
3. 在 CF 面板添加一个路由 Route 到端口（这个步骤会自动把二级域名加号，切记此时这个二级域名不能是已经存在的，否则就会失败）

我稍微优化了一下第二步，采用 docker-compose 的方法
1. 把面板给的 `docker` 版指令里面的 `tunnel` 命令部分复制出来。比如 `docker run cloudflare/cloudflared:latest tunnel --no-autoupdate run --token XXXXXXX `
2. 写一个 docker-compose.yml 文件，在 command 那里替换掉（主要是 token）注意这里要加 networks 且必须是external的
```
services:
    cloudflared:
        container_name: cloudflared
        restart: unless-stopped
        image: cloudflare/cloudflared:latest
        command: tunnel --no-autoupdate run --token XXXXXXX
        networks:
          - cf_tunnel

networks:
  cf_tunnel:
      external: true
```
3. 执行`docker network create cf_tunnel`
4. 执行`docke compose up -d` 
5. 然后去其他的服务容器修改他们的yml