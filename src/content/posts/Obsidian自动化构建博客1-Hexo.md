---
title: Obsidian自动化构建博客1-Hexo
tags:
  - Hexo
  - Obsidian
  - Workflow
category: 技术
description: 本文分享如何搭建一套源码私有化、编译云端化、写作原生化的个人博客系统，实现「本地只管写，云端自动发」的极致体验。
published: 2026-03-19T17:00:00+08:00
draft: false
image: https://image.heavenroad.org/Cover.png
---
link test:
[Obsidian自动化构建博客8-最终章](/posts/obsidian自动化构建博客8-最终章/)

## Obsidian 自动化构建博客 1。梦开始的地方

> [!TIP]
> Obsidian + Git 使用了一段时间非常丝滑，想着折腾一下 Github Pages 或许能直接从 Obsidian 发布到一个静态博客？NS 上看到有个 Hexo+butterfly 主题很漂亮，于是就开始折腾。

> [!NOTE]
> Obsidian Git - Submodule - Github Repo - Github Action - Github Public Repo - Cloudflare Page

### 1。架构设计

- **Obsidian**：作为全能笔记库，通过 **Git Submodule** 嵌入博客源码。
- **GitHub 私有仓库**：存放 Hexo 配置文件及 Markdown 原稿，确保创作隐私。
- **GitHub Actions (CI/CD)**：充当云端大脑，监听源码变动并自动执行环境安装、编译与发布。
- **GitHub 公共仓库**：作为托管载体，仅存放编译后的静态网页文件。

---

### 2。自动化部署配置 (CI/CD)

#### 2.1 建立 SSH 信任链
为了让 GitHub Actions 自动向公共仓库推送代码，需配置 SSH 密钥：

1.  **生成密钥对**：`ssh-keygen -t rsa -b 4096 -f github-deploy-key`。
2.  **公共仓库设置**：进入 `username.github.io` > **Settings** > **Deploy keys**，添加公钥内容，并勾选 **Allow write access**。
3.  **私有仓库设置**：进入 `hexo-blog-source` > **Settings** > **Secrets**，添加私钥内容，命名为 `HEXO_DEPLOY_PRI`。

#### 2.2 编写自动化脚本
在私有仓库根目录下创建 `.github/workflows/deploy.yml`：

```yaml
name: Hexo Auto Deploy
on:
  push:
    branches: [main]

env:
  FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source
        uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '24'
          cache: 'npm'
      - name: Configure SSH
        run: |
          mkdir -p ~/.ssh/
          echo "${{ secrets.HEXO_DEPLOY_PRI }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
      - name: Deploy
        run: |
          npm install -g hexo-cli
          npm install
          hexo clean && hexo g -d
```

---

### 3。深度集成 Obsidian

#### 3.1 引入子模块
将博客源码仓库作为子模块嵌入你的 Obsidian 库中：

```bash
cd ~/your_obsidian_vault
git submodule add https://github.com/yourname/hexo-blog-source.git GitBlog
```

#### 3.2 视觉净化 (File Explorer++)
为保持侧边栏纯净，仅显示文章目录，在 **File Explorer++** 插件的 **Hide Filter** 中设置以下正则：

```regex
^GitBlog/(?!(source/|source$)).*|^GitBlog/source/(?!(_posts/|_posts$)).*
```

#### 3.3 其他配置 (可选)
- Templater：配置给 _posts 下的文件自动加 `post front-matter`
- Git：开启 **Submodule management**
- Linter：把路径添加到 **Folders to ignore**
- Files and links：把路径添加到 **Advanced - Excluded Files**

---

### 4。极致写作流体验

1.  **撰写**：在 Obsidian 的 `GitBlog/source/_posts` 目录下直接新建 Markdown。
2.  **提交**：保存后 **Obsidian Git** 插件自动进行 Push。
3.  **生效**：GitHub Actions 接收信号，自动在云端完成部署。

> 只要配置一次，余下的只有创作。

### 5。**Cloudflare Pages** 套 CDN + 自定义域名

确保 Cloudflare 获得 Github 授权后，可以将静态仓库设为 **Private**。

1.  **权限授予**：在 GitHub 设置中，为 Cloudflare Pages App 开启对私有静态仓库的访问权限。
2.  **新建 Pages**
	- 入口：Workers & Page - Create Application 页面的底部中间一行小字：*"Looking to deploy Pages？Get started"*
    - **Framework preset**: `None`
    - **Build command**：(保持为空)
    - **Build output directory**: `/`
3.  **自动触发**：只要 GitHub Actions 完成推送，Cloudflare 就会在 30 秒内完成全球 CDN 同步。
4. 部署完成后，就可以新建 *Custom Domains* 了。

---

#### 避坑

**修复时区解析错误**
若 Actions 报错 `TypeError: Cannot read properties of null (reading 'utcOffset')`，通常是由于未指定时区导致日期解析失败。
在 `_config.yml` 中明确指定：
```yaml
timezone: Asia/Shanghai
```

**hexo-abbrlink**
为了避免中文标题生成的 URL 乱码，使用哈希值。
1.  **安装插件**：`npm install hexo-abbrlink --save`
2.  **配置 `_config.yml`**：
    ```yaml
    permalink: posts/:abbrlink/
    abbrlink:
      alg: crc32 # 算法：crc16 或 crc32
      rep: hex   # 进制：dec (数字) 或 hex (十六进制)
    ```
