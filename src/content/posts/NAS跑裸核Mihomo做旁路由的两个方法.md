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
Last Modified: 2026-07-29 14:07:07
---

假如局域网 LAN 中有一台 NAS 或者闲置的主机可以，那么就可以获得一个 24 小时在线的旁路由，给所有设备翻墙使用。


## 方法一：用Docker容器跑

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

SSH命令行用 `docker compose up -d` 启动，或者在群晖DSM的Container Manager中添加后启动。（不会的问AI）

## 方法二：直接跑二进制

在官方发布页`https://github.com/MetaCubeX/mihomo/releases/`下载自己机器架构的（群晖一般来说下`mihomo-linux-amd64-****.gz`)，解压到NAS。以下步骤需要通过SSH执行（不会的问AI）

### 准备工作

把解压的二进制文件改好名字，和配置文件找一个空目录存放，比如`/vol/mihomo/bin` 和 `~/mihomo/config`

### 1. 创建 systemd 服务配置文件

创建 service 文件：


```
sudo vim /etc/systemd/system/mihomo.service
```

在文件中写入以下内容：

```
[Unit]
Description=Mihomo Daemon Service
After=network.target network-online.target nss-lookup.target
Wants=network-online.target

[Service]
Type=simple
User=root
ExecStart=~/mihomo/bin/mihomo -d ~/mihomo/config
Restart=on-failure
RestartSec=5s
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

> **参数说明：**
> 
> - **`Restart=on-failure`**：当程序异常退出（crash、非 0 状态码退出）时会自动重启；如果是手动正常停止则不会重启。
>     
> - **`RestartSec=5s`**：崩溃后等待 5 秒再自动重启，避免频繁死循环重启。
>     
> - **`LimitNOFILE=65535`**：解除文件句柄数限制，防止网络并发较高时报错。
>     

---

### 2. 重新加载配置并启动服务

保存并退出编辑器后，运行以下命令：

Bash

```
# 1. 重新加载 systemd 配置
sudo systemctl daemon-reload

# 2. 设置开机自启
sudo systemctl enable mihomo

# 3. 立即启动服务
sudo systemctl start mihomo
```

---

### 3. 服务状态管理与日志查看

- **查看服务运行状态**：
    
    Bash
    
    ```
    sudo systemctl status mihomo
    ```
    
- **查看运行日志（实时监控）**：
    
    Bash
    
    ```
    sudo journalctl -u mihomo -f
    ```
    
- **手动停止服务**：
    
    Bash
    
    ```
    sudo systemctl stop mihomo
    ```
    
- **手动重启服务**：
    
    Bash
    
    ```
    sudo systemctl restart mihomo
    ```

然后浏览器访问 `https://metacubex.github.io/metacubexd` endpoint 的 ip 填入当前机器的 IP 前面加 HTTP （比如`http://192.168.0.100`)，端口默认`9090` 应该就可以进行控制了，非常方便。比部署虚拟机旁路由占用资源低多了。
