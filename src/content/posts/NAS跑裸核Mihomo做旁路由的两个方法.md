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
Last Modified: 2026-07-29 19:07:66
---

NAS 虚拟 OPENWRT 其实要配置的东西太多了，比如 ipv6 就可能劝退一批人，但是裸核跑比想象的方便多了，系统占用极低，和 Openwrt 对比，内存只要 50MB，磁盘空间也只要一个核。前提条件是配置文件需要手搓一个完美或者找一个优秀的模版，把 dns、分流规则等都写好，有这些之后就可以开始了。（不懂的问 AI）

> 旁路由开启 tun 后可以直接把家里的 DHCP 配置改成让 NAS 当网关，就可以做到全屋自动生效，手机平板电脑 TV 等等啥都不用装。只要在 NAS 中关闭 DHCP，手动配置网卡，网关用光猫/交换机/路由器的 IP。

## 方法一：用 Docker 容器跑

在群晖 DSM 的 Container Manager 中添加一个 project，填入以下内容，并且把订阅或者自建的配置文件存成 `config.yaml` 放在同一个目录下

```
services:
  mihomo:
    image: docker.1ms.run/metacubex/mihomo:latest
    container_name: mihomo
    restart: unless-stopped
    # Host network mode (Required for TUN / Transparent Proxying)
    network_mode: host
    # Enhance system open file limits to prevent "too many open files" errors under heavy traffic
    ulimits:
      nofile:
        soft: 65535
        hard: 65535
    # CPU priority tuning
    cpu_shares: 2048

    # Environment variables
    environment:
      - TZ=Asia/Shanghai  # Adjust to your local timezone for correct log timestamps

    # Network permissions for TUN mode
    cap_add:
      - NET_ADMIN
      - NET_RAW
    devices:
      - /dev/net/tun:/dev/net/tun

    # Mount entire directory to retain GeoIP data, rule-providers, and runtime cache
    volumes:
      - .:/root/.config/mihomo

    # Prevent Synology disk space exhaustion from Docker log accumulation
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

> 容器配置中启用了可以使用 tun 模式，但是真正的是否开启，还是要看配置文件里面是怎么写的。也可以在浏览器面板中手动开启关闭（见最后）

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
