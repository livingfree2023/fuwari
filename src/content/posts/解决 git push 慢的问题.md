---
title: 解决Git push慢的问题
category: Blog
tags:
  - GitBlog
published: 2026-07-10T10:07:76+08:00
image: https://image.heavenroad.org/default_cover.webp
slug: slug20260710100776
upload: false
Last Modified: 2026-07-10 10:07:60
---

git push 等操作打开代理 tunnel 模式时正常，但是不开 tunnel 就没法用或者速度极慢

即使在 git config 中配置了 http + https proxy 也没用。

原因是，如果 remote 地址是 `git@xxx.github.com` 的话，其实 git 底层走的是 ssh 通道，所以 `git config` 的配置被忽略了。这是能够起作用的配置是 `~/.ssh/config `, 在里面添加

```
Host *.github.com    
    ProxyCommand nc -X5 -x 127.0.0.1:端口 %h %p
```

**尽量不要用 localhost，直接用 127 地址，避免被解析 v6 节外生枝**
