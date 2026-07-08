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
Last Modified: 2026-07-08 09:07:11
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
2. 写一个 docker-compose.yml 文件，在 command 那里替换掉（主要是 token）注意这里要加 networks 且必须是 external 的
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
3. 先执行 `docker network create cf_tunnel` **必须，最后解释** [^1]
4. 再执行 `docke compose up -d`
5. 然后去其他的服务容器修改他们的 yml 文件，让他们全部加到同一个网络中。
```
services:
  my_web_nginx:
    image: nginx:latest
    container_name: my_nginx  # 重点：固定容器名
    restart: always
    networks:
      - cf_tunnel             # 将容器加入 cf_tunnel 网络
    # 注意：这里甚至不需要暴露 ports (如 80:80)，因为 cloudflared 在内部就能访问它，更安全

  my_blog:
    image: wordpress:latest
    container_name: my_blog   # 重点：固定容器名
    restart: always
    networks:
      - cf_tunnel             # 同样加入 cf_tunnel 网络

networks:
  cf_tunnel:
    external: true            # 声明这是一个外部已存在的网络
```

6. 当所有容器都启动并加入到 `cf_tunnel` 网络后，`cloudflared` 容器就可以通过**其他容器的 container_name + 容器内端口**直接访问它们了，不需要经过宿主机的 IP。打开 Cloudflare Zero Trust 控制台 (Tunnels 页面)，在你的 Tunnel 规则中添加 **Public Hostname**， 比如 `nginx.yourdomain.com` 映射 `http://my_nginx:80`。

这里最大的好处就是容器不再暴露服务端口，也不用配置端口映射了，而且可以直接使用容器的名字。非常安全又非常方便。

[^1]: 网络不能是由 `cloudflared` 的 Compose 项目创建的，否则意味着：

	- **必须**先启动 `cloudflared`，其他容器才能启动。
	- 命名会被自动加上“前缀”
	- 对 `cloudflared` 执行 `docker compose down` 时，Compose 会尝试**删除**这个网络。如果此时其他容器正挂在这个网络上，`docker compose down` 就会报错，提示网络正在被使用无法删除。
