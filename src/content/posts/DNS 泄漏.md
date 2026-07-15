---
Last Modified: 2026-07-15 11:07:41
title: DNS 泄漏
category: Tech
tags:
  - DNS
  - VPS
published: 2026-04-12T11:04:65+08:00
image: https://image.heavenroad.org/default_cover.webp
slug: slug20260412110465
upload: false
---

首先你要搞清楚 dns 泄漏风险的定义，别过度惊慌了。泄露也分两种，外国人说 dns leak 主要是防止你访问的网站通过 dns 探测发现你的 ip 所在的国家是伪装的。中国人说的 dns 泄漏主要是防止 ISP/网帽发现。这两种前一种靠分流防范，后一种靠加密防范。

DNS 普通请求不加密，敏感域名 DNS 请求用 udp 传在公网上，ISP 可以看啊哦，不管你用国内还是国外的 dns。但用 DoH/TLS 等加密后的请求 ISP 看不见，代价是速度慢，加密本身和 tcp 都增加延迟。

但是非敏感域名就算泄漏没关系 - 比如你访问 github,google,youtube, instagram 这些域名其实并不是很敏感，大把不会翻 q 的用户好奇想看看，很多 app 或者网页的 js 库之类的会底层去连，太多了 false alarm 帽子根本没空去查，更别说访问你自己的小鸡域名。因为 dns 请求不代表你干了什么，国内采用污染的方式解决就是懒得理了。

真要追求洁癖配置，参考下面这些去配置，先通过普通 DNS 拿到 DoH 和节点 IP，再通过 DoH/TLS 请求 DNS。

default-nameserver:
>   用途：解析 DNS 服务器本身的域名。
>   如果你在 nameserver 中使用了 DoH（如 https://dns.pub/dns-query），
>   系统首先需要知道 dns.pub 的 IP。由于此时 Clash 内部 DNS 还未完全启动，
>   必须依靠 default-nameserver。
>   这里只能填写 纯 IP 地址（或者简单的 DoH/DoT IP），不能填写依赖域名解析的地址。

proxy-server-nameserver:
>   如果你的节点配置里 server 填的是域名（如 hk01.example.com），
>   系统需要解析这个域名才能建立连接。
>   使用此字段可以确保节点解析速度最快，且不被普通的 DNS 规则干扰。
>   如果留空，它会默认使用 nameserver 的配置。  

direct-nameserver:
>   专门用于 DIRECT（DIRECT）出站 的域名解析。
>   这是 Meta 内核特有的字段。当流量命中 DIRECT 规则时，
>   系统会优先调用这里的 DNS 查 IP。
>   它的存在是为了防止：因为主 DNS 开启了特殊设置（如分流），
>   导致本该 DIRECT 的国内网站被解析到了错误的国外 IP，从而影响访问速度。

nameserver:
>   核心解析器，用于处理大部分普通的 DNS 请求。
>   它是主 DNS 列表。当一个请求进来且不匹配 nameserver-policy 时，会通过这里列出的服务器进行解析。
>   通常配置国内低延迟的 DNS（如阿里、腾讯）。
>   在 fake-ip 模式下，它决定了 Clash 内部处理流量时的解析逻辑。

fallback:
>   物理位置在境外且没有污染，确实可以不再需要传统意义上的 fallback 组，如果 nameserver 是 adg 就不用配置了
>   如果客户端装了 adg，dns 就不需要 adg，只要在 fallback 用就行
>   处理国外域名或被污染的解析。
>   它是为了解决 DNS 污染 而设计的。
>   判定机制：通常与 fallback-filter 配合使用。如果 nameserver 解析出来的 IP 
>   不在 geoip-code（通常是 CN）范围内，或者属于 ipcidr 黑名单，
>   Clash 就会采用 fallback 的解析结果。
>   通常配置国外的加密 DNS（DoH/DoT），如 Google 或 Cloudflare。

fallback-filter:
>   访问以下域名是会使用 fallback 解析。
>   它的作用是确保某些特定的域名，一定会走 fallback 组进行解析。
>   这样可以避免这些域名被污染，确保它们能被正确解析到国外 IP。

但我发现 clash meta 为了追求速度，会并发到所有的 nameserver 和 fallback-server，你要避免 ISP 泄漏的话，确保这 2 个地方只填写 DoH/TLS。

折腾明白之后还可以自己在节点搭建 adguard DoH 服务自己解析
