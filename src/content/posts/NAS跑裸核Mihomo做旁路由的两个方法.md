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
Last Modified: 2026-07-31 07:07:88
---

NAS 虚拟 OPENWRT 其实要配置的东西太多了，比如 ipv6 就可能劝退一批人，但是裸核跑比想象的方便多了，系统占用极低，和 Openwrt 对比，内存只要 50MB，磁盘空间也只要一个核。前提条件是配置文件需要手搓一个完美或者找一个优秀的模版，把 dns、分流规则等都写好，有这些之后就可以开始了。（不懂的问 AI）

> 旁路由开启 tun 后可以做到全屋自动生效，手机平板电脑 TV 等等设备都不需要装梯子翻墙。简单的说操作方法就是：
> 1. 装好旁路由
> 2. 家里的 路由器下发的 DHCP 配置中的网关 IP 改成旁路由的 IP
> 	1. 如果 NAS 上直接跑裸核或者在容器里直接跑裸核，那就填 NAS 的 IP
> 	2. 如果用了 MacVLan 给容器指定了别的 IP，那就填 mihomo 所在容器的 IP）
> 3. 特别重要：在 NAS 的网卡设置中手动配置 IP，网关 IP 要填写光猫/交换机/路由器的 IP。（否则就环路了，啥都出不去）

## 方法一：用 Docker 容器跑

![](Pasted%20image%2020260731075808.png)

> Docker容器跑裸核分tun和非tun模式，又分是否支持ipv6的情况，如果要

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

---

## 更新：
如果要开启 tun，又不影响 dsm 本身的服务（比如 pt），用 macvlan 把容器换一个 IP。但是要解决 IPv6 的问题（IPv6 需要能支持光猫换前缀的情况）。解决后又会发现无论如何 tun 无法启用，折腾一整天，最后 Gemini 总结如下：

在 Synology DSM 7.x 系统上通过 Docker 部署 Mihomo (Clash Meta) 旁路由时，**Macvlan 网络架构** 结合 **TUN 模式** 是兼顾网络性能与全协议（包含 ICMP / UDP）代理的最佳实践。

然而，由于群晖 Linux 内核（5.10+）对动态创建虚拟网卡的默认安全限制，直接开启 TUN 模式经常会遇到 `Start TUN listening error: configure tun interface: permission denied` 或 `RTNETLINK answers: Permission denied` 报错。

本文记录了完整的排查过程与最终的生产级配置方案，包含 IPv6 双栈动态路由与全局透明代理的完美结合。

## 1. 架构与痛点解析

### 核心痛点：为什么 TUN 会在群晖 DSM 下报 `Permission denied`？

在 Macvlan 模式下部署容器并启动 TUN，Mihomo 初始化时会按顺序执行以下动作：

1. 创建虚拟网卡（例如 `Meta` / `tun0`）。

2. 向该网卡绑定 IPv4 / IPv6 地址（默认私有段为 `198.18.0.1/30` 和 `fdfe:dcba:9876::1/126`）。

3. 拉起网卡并注入 `auto-route` 路由表。

**根本病灶**：群晖 DSM 内核在动态创建新的网络接口时，**默认会开启 `disable_ipv6=1`**。当 Mihomo 试图为新生成的 `Meta` 网卡赋予私有 IPv6 地址时，系统内核会直接拦截并抛出 `RTNETLINK answers: Permission denied`，最终导致 Mihomo 启动崩溃。

### 解决思路

通过 Docker `sysctls` 与启动脚本联合解锁内核对动态接口的 IPv6 限制，同时解除 Macvlan 环境下的路由环路与 DNS 绑定端口冲突。

## 2. 宿主机前期准备 (Synology DSM)

必须确保群晖宿主机加载了 `tun` 内核模块并挂载了设备节点。

### 1. 检查 `tun` 模块状态

通过 SSH 登录群晖宿主机执行：

Bash

```
ls -l /dev/net/tun
```

若提示文件不存在或未加载，手动建立节点与赋予权限：

Bash

```
sudo insmod /lib/modules/tun.ko 2>/dev/null || true
sudo mkdir -p /dev/net
sudo mknod /dev/net/tun c 10 200 2>/dev/null || true
sudo chmod 0666 /dev/net/tun
```

### 2. 设置群晖开机自动加载 TUN

在群晖 **控制面板** -> **任务计划** 中新增一个 **触发的任务 (用户定义的脚本)**：

- **用户**：`root`

- **事件**：开机 (Boot-up)

- **任务步骤脚本**：

Bash

```
if [ ! -c /dev/net/tun ]; then
    insmod /lib/modules/tun.ko
    mkdir -p /dev/net
    mknod /dev/net/tun c 10 200
    chmod 0666 /dev/net/tun
fi
```

## 3. 项目最终配置文件

基于 Mihomo 官方 **Alpine** 镜像，无需任何自定义 Dockerfile，全部由 Compose 与启动脚本托管。

### 文件结构

Plaintext

```
/volume2/docker/vlan-mihomo/
├── compose.yaml
├── entrypoint-v6.sh
└── config.yaml
```

### 配置一：`compose.yaml`

YAML

```
services:
  metacubexd:
    image: docker.1ms.run/metacubex/mihomo:latest
    container_name: vlan_metacubexd
    restart: unless-stopped
    environment:
      TZ: '${TZ:-Asia/Shanghai}'
    ulimits:
      nofile:
        soft: 65535
        hard: 65535
    cpu_shares: 2048

    volumes:
      - '.:/root/.config/mihomo'
      - './entrypoint-v6.sh:/entrypoint-v6.sh:ro'

    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://127.0.0.1:9090"]
      interval: 30s
      timeout: 5s
      start_period: 10s
      retries: 3

    # 关键：通过 sysctls 允许容器内核为新创接口开启 IPv6 并放行路由转发
    sysctls:
      - net.ipv6.conf.all.disable_ipv6=0
      - net.ipv6.conf.default.disable_ipv6=0
      - net.ipv4.ip_forward=1
      - net.ipv6.conf.all.forwarding=1
      - net.ipv6.conf.all.accept_ra=2
      - net.ipv6.conf.eth0.accept_ra=2
      - net.ipv6.conf.eth0.accept_ra_pinfo=2
      - net.ipv6.conf.eth0.accept_ra_defrtr=0
      - net.ipv6.conf.eth0.autoconf=1

    cap_add:
      - NET_ADMIN
      - NET_RAW
      - SYS_ADMIN

    security_opt:
      - seccomp:unconfined

    devices:
      - /dev/net/tun:/dev/net/tun

    networks:
      macvlan_net:
        ipv4_address: 192.168.0.2

    privileged: true
    entrypoint: ["/bin/sh", "/entrypoint-v6.sh"]

networks:
  macvlan_net:
    external: true
```

### 配置二：`entrypoint-v6.sh`

Bash

```
#!/bin/sh
set -e

export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:$PATH"

# 1. 强制解锁内核对新建接口（如 Meta/tun0）的 IPv6 限制
sysctl -w net.ipv6.conf.all.disable_ipv6=0 2>/dev/null || true
sysctl -w net.ipv6.conf.default.disable_ipv6=0 2>/dev/null || true

# 2. 检查并动态分配 eth0 的 IPv6 前缀地址（如未绑定则静态追加）
DYNAMIC_ADDR=$(ip -6 addr show eth0 scope global | grep 'inet6 240' | head -n1 | awk '{print $2}')
if [ -z "$DYNAMIC_ADDR" ]; then
    echo "[IPv6] Adding static IPv6 address to eth0..."
    ip -6 addr add 2409:8a28:12eb:6e10::2/64 dev eth0 2>/dev/null || true
fi

# 3. 修正并重置 IPv6 默认网关路由
echo "[IPv6] Setting up IPv6 default route..."
ip -6 route del default 2>/dev/null || true
ip -6 route add default via fe80::f66d:2fff:fee6:fab5 dev eth0 2>/dev/null || true

# 4. 覆写容器内部 DNS 解析，防止启动初期依赖宿主机 DNS 超时
echo "nameserver 119.29.29.29" > /etc/resolv.conf
echo "nameserver 223.5.5.5" >> /etc/resolv.conf

# 5. 启动 Mihomo 主程序
echo "[Mihomo] Starting Mihomo daemon..."
exec /mihomo -d /root/.config/mihomo
```

*权限说明：创建脚本后请运行 `chmod +x entrypoint-v6.sh`。*

### 配置三：`config.yaml` 核心段落

YAML

```
# 指定物理出网接口，防止 Macvlan 模式下 TUN 出口流量回流造成无限环路
interface-name: eth0

external-controller: 0.0.0.0:9090
secret: hello@world
mixed-port: 1080
allow-lan: true
bind-address: '*'
ipv6: true
mode: rule
log-level: error

# TUN 模式核心参数
tun:
  enable: true
  stack: gvisor       # 使用用户态网络栈，避开群晖内核高级参数修改限制
  auto-route: true
  strict-route: false # 必须为 false，防止强制修改宿主机主表
  auto-detect-interface: false # 在 Macvlan 下必须关闭自动识别，防止错误绑定
  inet4-address: 198.18.0.1/30
  inet6-address: fdfe:dcba:9876::1/126
  dns-hijack:
    - 'any:53'
    - 'tcp://any:53'

# 内置 DNS 服务参数（为局域网提供 DNS 劫持与 Fake-IP 解析）
dns:
  enable: true
  listen: 0.0.0.0:53  # 监听 53 端口，支持作为局域网主 DNS 使用
  ipv6: true
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16
  respect-rules: false
  use-hosts: true
  nameserver:
    - 119.29.29.29
    - 223.5.5.5
    - https://doh.pub/dns-query
```

## 4. 部署与功能验证

在项目目录下执行容器一键启动：

Bash

```
docker compose up -d --force-recreate
```

### 1. 验证 TUN 接口与网络状态

进入容器查看网络设备：

Bash

```
docker exec -it vlan_metacubexd sh -c "ip a"
```

**成功标志**：

- 出现 `Meta` 虚拟接口且状态为 `UP,LOWER_UP`。

- `Meta` 接口成功绑定了 IPv4 (`198.18.0.1/30`) 与 IPv6 (`fdfe:dcba:9876::1/126`)。

- `eth0` 正常拿到公网 IPv6 地址 (`2409:…/64`)。

### 2. 验证容器内双栈连通性与 DNS 状态

Bash

```
# 验证 IPv4 连通性
docker exec -it vlan_metacubexd ping -c 2 baidu.com

# 验证 IPv6 连通性
docker exec -it vlan_metacubexd ping6 -c 2 2a0e:97c0:3f0:1::1d1d
```

### 3. 验证局域网客户端接入

在同局域网的电脑/手机上，将**网关**与 **DNS 服务器** 手动指定为 `192.168.0.2`：

Bash

```
# 在客户端命令行验证 DNS 解析
nslookup sony.com 192.168.0.2
```

此时解析应瞬间返回 Fake-IP（`198.18.x.x`）或正确 IP，全网 TCP / UDP / ICMP 流量将无感通过 Mihomo TUN 模式进行透明代理转发。
