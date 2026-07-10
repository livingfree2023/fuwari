---
title: 解决Git push慢的问题
category: Blog
tags:
  - GitBlog
published: 2026-07-10T10:07:76+08:00
image: https://image.heavenroad.org/default_cover.webp
slug: slug20260710100776
upload: false
Last Modified: 2026-07-10 10:07:79
---
## 问题

`git push` 等操作打开代理 tunnel 模式时正常，但是不开 tunnel 就没法用或者速度极慢

即使在 git config 中配置了 http + https proxy 也没用。

## 原因

如果 remote 地址是 `git@xxx.github.com` 的话，其实 git 底层走的是 ssh 通道，所以 `git config` 的配置被忽略了。git 配置中的 https proxy 只对使用 https 链接的 repo 起作用。`git remove -v` 查看当前 repo 的链接形式。

## 方案 A

能够对 ssh 起作用的配置是 `~/.ssh/config `, 在里面添加

```
Host *.github.com    
    ProxyCommand nc -X5 -x 127.0.0.1:端口 %h %p
```

**尽量不要用 localhost，直接用 127 地址，避免被解析 v6 节外生枝**
**用 `*.github.com` 可以对 `gist.github.com` 生效**

## 方案 B

配置 git config 中的 ssh 也可以起到类似作用

```
git config --global core.sshCommand "ssh -o ProxyCommand='nc -X connect -x 127.0.0.1:端口 %h %p'" 
```

**缺点是这样本机所有的 git 指令都会走代理，如果工作区域有非 github 的 repo，可能不够灵活。但如果配置好代理分流的话也无所谓，看个人喜好**
