---
title: 64M Alpine 小内存VPS可以用的脚本
category: Blog
tags:
  - GitBlog
  - alpine
  - Singbox
  - XTLS
  - Realm
published: 2026-05-21T08:05:07+08:00
image: https://image.heavenroad.org/default_cover.webp
slug: slug20260521080507
upload: false
Last Modified: 2026-05-24 08:05:58
---
Xray NOKEY
```
curl -fsSL -o /usr/local/bin/nokey https://raw.githubusercontent.com/livingfree2023/nokey/refs/heads/main/nokey.sh && chmod +x /usr/local/bin/nokey && nokey
```

Install SB
```
bash -c "$(curl -fsSL https://raw.githubusercontent.com/caigouzi121380/singbox-deploy/main/install-singbox-yyds.sh)"
```

Realm
```
wget -qO- https://raw.githubusercontent.com/zywe03/realm-xwPF/main/xwPF.sh |  bash -s install
```
