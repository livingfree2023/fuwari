---
title: Obsidian自动化构建博客7-简化
category: 技术
tags:
  - GitBlog
  - Obsidian
published: 2026-03-22T20:37:00+08:00
draft: false
image: https://image.heavenroad.org/resources/Pasted%20image%2020260322205640.png
---

> [!TIP]
> 我用了两天感觉还是不妥，因为怕被墙，所以服务器还是套了 CF CDN，看了下 CF 的节点分配的是西雅图，好家伙，请求绕美国再到亚洲，恢复也要绕美国再回中国。一个请求要飞四趟太平洋…… 于是想了个新方法，Vault 里面加 action 监控 blog 目录，把变动 push 到一个新的 fuwari 仓库。然后用 CF Page 去编译然后直接呈现。整个过程极简，也不需要本地有环境了。更关键的是手机 Obsidian+GitSync 也可以编辑和触发了！

> [!NOTE]
> Obsidian Git Push **Vault** - **Vault** Action 监控 Push 到 **Fuwari** - Cloudflare Pages 自动编译同时发布。
> 本地插件可以删除 Shell Commands

## 核心架构

1. **私有仓库 (Source)**：存放完整的 Obsidian Vault。
    
2. **网站仓库 (Site)**：存放 Astro (Fuwari) 模板代码。
    
3. **GitHub Action**：监听私有库变动，仅将指定的博客文件夹推送到网站仓库。
    
4. **Cloudflare Pages**：监测网站仓库变动，自动构建并发布静态页面。

---

## 第一步：获取 GitHub 访问令牌 (PAT)

为了让私有仓库有权“写”数据到网站仓库，需要生成一个 **Fine-grained Personal Access Token**：

1. **Repository selection**: 仅选择你的 `网站仓库`。
    
2. **Permissions**:
    
    - `Contents`: 选择 **Read and write** (核心权限)。
        
    - `Metadata`: 选择 **Read-only**。
        
3. **生成并保存**：将生成的 Token 存入私有仓库的 **Settings > Secrets and variables > Actions**，命名为 `BLOG_SYNC_TOKEN`。

---

## 第二步：配置自动同步工作流

在私有 Obsidian 仓库根目录下创建 `.github/workflows/deploy.yml` 文件：

YAML

```
name: Sync Blog Folder to Astro
on:
  push:
    paths:
      - '你的博客文件夹/**' # 仅当此文件夹内的文件变动时触发

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Vault
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Push to Site Repo
        uses: cpina/github-action-push-to-another-repository@main
        env:
          API_TOKEN_GITHUB: ${{ secrets.BLOG_SYNC_TOKEN }}
        with:
          source-directory: '你的博客文件夹' # 源文件夹名称（严格区分大小写）
          destination-github-username: '你的用户名'
          destination-repository-name: '网站仓库名'
          target-directory: 'src/content/posts' # Astro 存放文章的标准路径
          user-email: github-actions[bot]@users.noreply.github.com
          target-branch: main
```

---

## 第三步：Cloudflare Pages 部署设置

1. 在 Cloudflare 仪表板选择 **Workers & Pages > Create application > Pages > Connect to Git**。
    
2. 选择你的 **网站仓库**。
    
3. **框架预设**：选择 `Astro`。
    
4. **构建设置**：
    
    - **Build command**: `npm run build`
        
    - **Build output directory**: `dist`
        
5. **保存并部署**。

---
