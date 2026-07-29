---
title: 用Docker容器跑裸核Mihomo做旁路由
category: Blog
tags:
  - Docker
  - Clash
  - Mihomo
  - 旁路由
published: 2026-07-29T12:53:03+08:00
image: https://image.heavenroad.org/default_cover.webp
slug: slug20260729125303
upload: false
Last Modified: 2026-07-29 13:07:96
---

假如局域网 LAN 中有一台 NAS 或者限制的主机可以跑 docker，那么就可以获得一个 24 小时在线的旁路由。

复制这个 `docker-compose.yml` 并且把 订阅或者自建的配置文件存成`config.yaml` 放在同一个目录下（端口映射按自己的配置调整）

``` 

services:
  mihomo:
    # docker.1ms.run 是国内加速用，不是必须
    image: docker.1ms.run/metacubex/mihomo:latest
    container_name: mihomo
    restart: always
    ports:
      - "1080:1080" # Mixed HTTP/SOCKS5 Proxy Port
      - "1071:1071" # HTTP
      - "1081:1081" # SOCKS
      - "9090:9090" # External Controller API Port (For Web UIs)
    volumes:
      - ./config.yaml:/root/.config/mihomo/config.yaml
```

如果是 linux，用 `docker compose up -d` 启动，其他 OS 问 AI。

然后浏览器访问 `https://metacubex.github.io/metacubexd` endpoint 的 ip 填入当前机器的 IP 前面加 HTTP （比如`http://192.168.0.100`)，端口默认`9090` 应该就可以进行控制了，非常方便。比部署虚拟机旁路由占用资源低多了。
