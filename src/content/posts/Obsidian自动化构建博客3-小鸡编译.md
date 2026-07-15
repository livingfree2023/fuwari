---
title: Obsidian自动化构建博客3-小鸡编译
category: 技术
description: ""
tags:
  - GitBlog
  - Obsidian
  - Workflow
published: 2026-03-20T10:00:00+08:00
draft: false
image: https://image.heavenroad.org/resources/Pasted%20image%2020260322210335.png
#slug: "20260319010003"
---

> [!TIP]
> 转念一想，瞎鸡鸡折腾什么 GitHub action 私有 repo 到公有 repo 再到 Cloudflare，我直接小鸡鸡上部署 Caddy + Cloudflared，在小鸡鸡上编译不就完事儿了么。开始折腾

> [!NOTE]
> Obsidian Git - Submodule - Github Repo - Github Action - VPS Build - Caddy - Cloudflare Tunnel

---

AI 夸夸美滋滋：这是一个非常成熟且高明的折衷方案。这种模式被称为 **"CI 触发 CD"**：让 GitHub Actions 负责**逻辑控制** (什么时候该部署)，让 VPS 负责**重体力劳动** (编译和静态托管)。

相比于在 VPS 上搞一套复杂的 Webhook，用 SSH 触发确实更简单、安全，且能复用你现有的 GitHub 工作流。

---

## 1。优化后的部署架构

这个架构避开了 GitHub Actions 编译慢的问题，同时解决了时区和 `draft: true` 的烦恼：

1. **GitHub Action**：仅作为一个 " 开关 "，通过 SSH 登录 VPS。
    
2. **Debian VPS**：拉取最新的 `source` 代码，在本地环境用 `pnpm` 快速增量编译。
    
3. **Caddy / Nginx**：在 VPS 上直接指向 `dist` 文件夹。
    
4. **Cloudflare Tunnel**：将穿透流量导向 VPS 本地的 80/443 端口。

---

## 2。更新你的 `deploy.yml`

你可以将原来的构建步骤全部删掉，改为一个极简的 SSH 触发脚本。请在 GitHub Secrets 中确保 `SSH_ID_RSA` 依然有效：

YAML

```
name: Deploy to VPS via SSH

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Execute remote commands via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.VPS_HOST }}      # 你的 VPS IP
          username: ${{ secrets.VPS_USER }}  # 你的 VPS 用户名 (如 root)
          key: ${{ secrets.SSH_ID_RSA }}     # 你的 SSH 私钥
          script: |
            cd /var/www/GitBlog
            git pull origin main
            # 这里的 pnpm 命令直接在 VPS 环境运行，速度极快
            pnpm install
            pnpm run build
            # 编译完后，Caddy 会自动识别 dist 里的新内容，无需重启
```

---

## 3。VPS 上的环境准备 (一次性配置)

在你的 Debian VPS 上，需要手动完成以下前置操作：

1. **克隆仓库**：

    Bash

    ```
    mkdir -p /var/www && cd /var/www
    git clone git@github.com:yourname/static-blog-source.git GitBlog
    ```
    
2. **安装 Node/pnpm**：

    Bash

    ```
    curl -fsSL https://get.pnpm.io/install.sh | sh -
    # 确保 Node 版本是 22+
    ```
    
3. **配置 Caddy (推荐)**：

    Caddy 是目前最简单的静态服务器，配置文件 `Caddyfile` 只需要两行：

    Code snippet

    ```
    :8080 {
        root * /var/www/GitBlog/dist
        file_server
    }
    ```
    
4. **Cloudflare Tunnel**：

    将隧道指向 `http://localhost:8080` 即可。

---

## 4。这种方案的巨大优势

- **真正的实时性**：VPS 上的 `pnpm` 拥有本地缓存，编译速度通常比 GitHub Actions 快 3-5 倍。
    
- **彻底解决时区问题**：只要你的 Debian 系统时间是正确的 (`Asia/Shanghai`)，Astro 编译时就会严格遵守你的时间，不再有 " 文章失踪 " 的情况。
    
- **调试极其方便**：如果编译报错，你可以直接连上 VPS 查看日志，甚至手动运行 `pnpm build` 复现问题。

---

## 5。避坑指南：SSH 权限

确保你的 VPS 能够**免密拉取 GitHub 仓库**。你需要在 VPS 上生成一对新的 SSH Key，并将公钥添加到 GitHub 仓库的 **Deploy Keys** 中。

这样 GitHub Action 触发 `git pull` 时，VPS 才能顺利拿到最新的代码。
