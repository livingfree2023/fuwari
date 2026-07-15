---
title: Obsidian自动化构建博客4-小鸡监控仓库
category: 技术
description: ""
tags:
  - GitBlog
  - Obsidian
  - Workflow
published: 2026-03-20T11:00:00+08:00
draft: false
#slug: "20260319010004"
image: https://image.heavenroad.org/resources/Pasted%20image%2020260322210153.png
---

> [!TIP]
> 我的整个 vault 本来都在 github 上，我其实完全可以放弃本地环境。在 vps 直接拉取我的 vault 但是只对里面的 GitBlog 目录进行编译，好像是个办法，开始折腾

> [!NOTE]
> Obsidian Git - Github Repo - Github Action - VPS Build - Caddy - Cloudflare Tunnel，发布过程减少了 2 个 repo

下面 AI 出场

---

这个思路非常清晰，本质上是把你的 VPS 变成了 " 云端工作站 "。既然你的整个 Obsidian Vault 已经在 GitHub 上，那么 VPS 就不再只是一个 Web 服务器，而是一个具备实时渲染能力的**镜像节点**。

由于 Fuwari (Astro) 的 `src` 目录必须包含特定的配置文件，而你的文章在 `GitBlog` 目录下，我们需要通过**软链接 (Symbolic Link)** 或**修改 Astro 配置**来实现 " 跨目录编译 "。

---

## 1。架构逻辑：双层目录结构

假设你的 Vault 路径是 `/var/www/my-vault`，而 Fuwari 模板安装在 `/var/www/fuwari-app`。

- **仓库 A (Vault)**：包含你的笔记、日记和 `GitBlog/posts`。
    
- **仓库 B (Fuwari)**：纯粹的程序代码。

**核心技巧：**我们把 Fuwari 的 `src/content/posts` 目录直接软链接到 Vault 里的 `GitBlog`。这样，每当 Vault 更新，Fuwari 看到的 " 源码 " 也就更新了。

---

## 2。VPS 一次性初始化脚本

在你的 Debian VPS 上执行以下操作：

Bash

```
# 1. 克隆整个 Obsidian Vault
cd /var/www
git clone git@github.com:yourname/my-vault.git

# 2. 克隆 Fuwari 模板（作为一个独立的干净环境）
git clone git@github.com:yourname/static-blog-source.git fuwari-app
cd fuwari-app
pnpm install

# 3. 建立软链接：让 Fuwari 的文章目录指向 Vault 里的目录
rm -rf src/content/posts
ln -s /var/www/my-vault/GitBlog src/content/posts
```

---

## 3。自动化部署脚本 (`deploy.sh`)

在 VPS 上创建一个脚本，由 GitHub Action 远程触发：

Bash

```
#!/bin/bash
# 进入 Vault 目录拉取最新笔记
cd /var/www/my-vault
git pull origin main

# 进入 Fuwari 目录进行编译
cd /var/www/fuwari-app
# 这里的编译会自动读取软链接过来的最新笔记
pnpm run build

# 此时 /var/www/fuwari-app/dist 就是最终的静态网页
```

---

## 4。这种方案的 3 个绝妙之处

1. **真正的 " 所见即所得 "**：你在手机或 iPad 上用 Obsidian 修改了 `GitBlog` 里的文件并 Push，VPS 收到指令后立刻编译。
    
2. **隔离生产环境**：你的 Vault 里可能有隐私笔记，但因为我们只软链接了 `GitBlog` 目录，Fuwari 编译时完全接触不到 Vault 里的其他私密文件夹，非常安全。
    
3. **极速增量更新**：由于 `pnpm` 缓存在 VPS 上，且 `node_modules` 不需要反复下载，整个编译过程通常在 10 秒内完成。

---

## 5。配合 Cloudflare Tunnel 反代

既然你用 Debian，建议直接用 **Caddy** 配合 Tunnel，这是目前最省心的组合：

**Caddyfile 配置：**

Code snippet

```
:8080 {
    root * /var/www/fuwari-app/dist
    file_server
    encode gzip
    handle_errors {
        rewrite * /404.html
        file_server
    }
}
```

**Tunnel 配置：**

将你的域名通过 Cloudflare 控制台指向 VPS 的 `http://localhost:8080`。

---

## 💡 给你的一个小提醒

- **图片路径**：如果你的 Obsidian 笔记里有图片，记得在 `GitBlog` 目录下建立一个 `assets` 文件夹。在 Fuwari 的 `src/content/config.ts` 中，确保图片引用路径是兼容的。
    
- **忽略文件**：在 `fuwari-app` 的 `.gitignore` 中，记得忽略 `dist` 和 `node_modules`。
    
