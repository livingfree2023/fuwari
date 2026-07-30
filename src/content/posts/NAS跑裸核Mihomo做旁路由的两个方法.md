---
title: NAS跑裸核Mihomo做旁路由的两个方法
category: Blog
tags:
  - Docker
  - Clash
  - Mihomo
  - 旁路由
  - NAS
published: 2026-07-29T12:53:03+08:00
image: https://image.heavenroad.org/default_cover.webp
slug: slug20260729125303
upload: false
Last Modified: 2026-07-30 08:07:48
---

NAS 虚拟 OPENWRT 其实要配置的东西太多了，比如 ipv6 就可能劝退一批人，但是裸核跑比想象的方便多了，系统占用极低，和 Openwrt 对比，内存只要 50MB，磁盘空间也只要一个核。前提条件是配置文件需要手搓一个完美或者找一个优秀的模版，把 dns、分流规则等都写好，有这些之后就可以开始了。（不懂的问 AI）

> 旁路由开启 tun 后可以直接把家里的 DHCP 配置改成让 NAS 当网关，就可以做到全屋自动生效，手机平板电脑 TV 等等啥都不用装。只要在 NAS 中关闭 DHCP，手动配置网卡，网关用光猫/交换机/路由器的 IP。

## 方法一：用 Docker 容器跑

在群晖 DSM 的 Container Manager 中添加一个 project，填入以下内容，并且把订阅或者自建的配置文件存成 `config.yaml` 放在同一个目录下

```
# All-in-one server example for a trusted LAN. Published ports bind to every
# host interface; do not expose them directly to the public internet. Use a
# firewall and a TLS reverse proxy for access outside the trusted network.
#
# Before `docker compose up -d`, create a `.env` file containing two different,
# strong random values:
#   CONTROL_TOKEN=...
#   CLASH_SECRET=...

services:
  metacubexd:
    image: ghcr.io/metacubex/metacubexd-server:latest
    container_name: metacubexd
    restart: unless-stopped
    environment:
      CONTROL_TOKEN: '${CONTROL_TOKEN:?Set CONTROL_TOKEN in .env}'
      CLASH_SECRET: '${CLASH_SECRET:?Set CLASH_SECRET in .env}'
      GITHUB_TOKEN: '${GITHUB_TOKEN:-}'
      # Optional: pre-fill the connect form's backend address (#2155).
      DEFAULT_BACKEND_URL: '${DEFAULT_BACKEND_URL:-}'
      CONTROL_PORT: '8080'
      CLASH_API_PORT: '9090'
      MIXED_PORT: '1080'
      TZ: '${TZ:-UTC}'
    ulimits:
      nofile:
        soft: 65535
        hard: 65535
    # CPU priority tuning
    cpu_shares: 2048

    volumes:
      - '.:/data'
      
    healthcheck:
      test: ['CMD', 'wget', '-qO-', 'http://127.0.0.1:8080/api/control/health']
      interval: 30s
      timeout: 5s
      start_period: 10s
      retries: 3
    #ports:
    #  - '8080:8080' # dashboard UI + /api/control agent API
    #  - '9090:9090' # Mihomo external-controller API + WebSocket
    #  - '1080:1080' # mixed HTTP/SOCKS proxy port
    #  - '1071:1071' # HTTP
    #  - '1081:1081' # socks        
    # TUN is an advanced Linux-only override. It grants network-administration
    # capability and makes the container share the host network namespace.
    # Remove `ports:` (Docker ignores it in host-network mode), uncomment the
    # three settings below, and enable `tun:` in the active Mihomo profile.
    network_mode: host
    privileged: true
    cap_add:
      - NET_ADMIN
      - NET_BIND_SERVICE
      - NET_RAW
    devices:
      - '/dev/net/tun:/dev/net/tun'
    
```

> 容器配置中启用了可以使用 tun 模式，但是真正的是否开启，还是要看配置文件里面是怎么写的。也可以在浏览器面板中手动开启关闭（见最后）

> tun 模式的大坑：
> 1. 开启 tun 一定要检查 dns 是否正确劫持，docker 中 dns-hijack 无论如何也没法生效，最后通过 dns 的 listen 设成 0.0.0.0:53 直接监听 53 才成功。
> 2. 注意 fake-ip 和 bt/pt 冲突，需要在 fake-ip-filter 中添加
>  - tracker.+
>   - geosite:category-stun
>   - geosite:category-pt
>   - PROCESS-NAME:qbittorrent-nox

这个方法可以全部图形化操作，具体步骤不赘述了（不会的问 AI）

## 方法二：直接跑二进制

> 以下步骤全部需要通过 SSH 命令行，（不会的建议使用方法一）

### 准备工作

下载自己机器架构的内核，解压到 NAS。（不知道下哪个的问 AI）

把解压的二进制文件改好名字，和配置文件找一个空目录存放，比如 `/volume1/mihomo/bin` 和 `/volume1/mihomo/config`

### 1. 创建 systemd 服务配置文件

创建 service 文件：

```
sudo vim /etc/systemd/system/mihomo.service
```

在文件中写入以下内容：（vim 不会用的问 AI）

```
[Unit]
Description=Mihomo Daemon Service
After=network.target network-online.target nss-lookup.target
Wants=network-online.target

[Service]
Type=simple
User=root
ExecStart=/volume1/mihomo/bin/mihomo -d /volume1/mihomo/config
Restart=on-failure
RestartSec=5s
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

> - **`Restart=on-failure`**：当程序异常退出（crash、非 0 状态码退出）时会自动重启；如果是手动正常停止则不会重启。>
> - **`RestartSec=5s`**：崩溃后等待 5 秒再自动重启，避免频繁死循环重启。
> - **`LimitNOFILE=65535`**：解除文件句柄数限制，防止网络并发较高时报错。

---

### 2. 重新加载配置并启动服务

保存并退出编辑器后，运行以下命令（只需要做一次之后重启关机都不用再做）：

```
# 1. 重新加载 systemd 配置
sudo systemctl daemon-reload

# 2. 设置开机自启
sudo systemctl enable mihomo

# 3. 立即启动服务
sudo systemctl start mihomo
```

---

### 3. 服务状态管理与日志查看（可选）

- **查看服务运行状态**：

```
sudo systemctl status mihomo
```

- **查看运行日志（实时监控）**：
```
sudo journalctl -u mihomo -f
```

- **手动停止服务**：
```
sudo systemctl stop mihomo
```

- **手动重启服务**：

```
sudo systemctl restart mihomo
```

## 图形化控制面板

无论用哪种方法部署后，用浏览器访问 `https://metacubex.github.io/metacubexd` endpoint url 的填入 NAS 的地址加 mihomo 的端口 ，比如`http://192.168.0.100:9090`，（端口默认`9090`） 应该就可以进行查看和控制了。比部署虚拟机搞 OpenWRT 做旁路由占用资源低多了，也便于管理。缺点是没有 OpenClash 之类的客户端自动更新订阅。

但是如果你在 DSM 的计划任务中新建一个自动更新订阅的脚本，定时执行，那就完全可以不用管了
示例：

```
(curl -fL -s "替换成订阅的链接"  -o /volume1/mihomo/config/config.yaml.tmp && mv /volume1/mihomo/config/config.yaml.tmp /volume1/mihomo/config/config.yaml) && echo "Successfully updated config.yaml" || echo "Failed to download config.yaml"
```

如果是用 docker 的，可以直接重启 docker 或者用面板手动 reload（不会的问 AI）
