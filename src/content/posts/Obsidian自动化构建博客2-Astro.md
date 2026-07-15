---
title: Obsidian自动化构建博客2-Astro
published: 2026-03-19T18:00:00+08:00
category: 技术
tags:
  - Astro
  - Workflow
  - Blog
draft: false
image: https://image.heavenroad.org/resources/Pasted%20image%2020260322210444.png
---

> [!TIP]
> Nodeseek 上看到有人说 Hexo 过时了，现在用 astro 更好。我一想，反正我写作是 markdown 啊，迁移成本应该远小于 ghost - wordpress 之类的，只要换个编译期就行了，于是开始折腾

> [!NOTE]
> Obsidian Git - Submodule - Github Repo - Github Action - Github Public Repo - Cloudflare Page 路径不变，Github Repo 换用 Astro Fuwari 的 fork

---

## 核心迁移策略：原地替换，内容至上

最稳妥的迁移方法是**原地替换**，即在原有的 Git 目录下，清理 Hexo 引擎，注入 Astro 引擎，但必须**确保文章源码 (Markdown) 绝对安全**。

### 迁移前的目录结构 (Hexo npm 模式)
```text
GitBlog/
├── .git/                <-- 必须保留，连接远程仓库
├── node_modules/        <-- 需清理
├── source/              <-- 核心内容，需备份
│   └── _posts/          <-- 所有文章 (.md)
├── themes/              <-- 需清理
├── _config.yml          <-- 需清理
└── package.json         <-- 需清理
```

---

## 实战演练：五步完成“换脑”手术

### Step 1：备份文章源码 (资产保全)
在进行任何删除操作前，先把文章拷贝到安全的地方：
```bash
# 假设你当前在父目录
mkdir -p ./temp_posts && cp -r ./GitBlog/source/_posts/* ./temp_posts/
```

### Step 2：清理旧引擎 (暴力拆解)
进入项目目录，除了 `.git` 文件夹，清理其余所有 Hexo 相关文件：
```bash
cd ~/obsidian_vault/GitBlog
# 慎重：确保你已经完成了上一步的备份！
rm -rf source themes scaffolds _config.yml _config.butterfly.yml package.json package-lock.json node_modules db.json
```

### Step 3：初始化 Astro (旁路注入)
由于 Astro 脚本默认不接受非空目录，我们需要先在一个临时文件夹里初始化，再搬运回来：
```bash
# 1. 回到父目录，在临时文件夹里初始化 Astro Blog 模板
cd ~/obsidian_vault
npm create astro@latest ./astro-temp -- --template blog --no-git --no-install

# 2. 将临时文件夹里的所有文件（包含隐藏文件）拷贝回 GitBlog
cp -r ./astro-temp/.* ./GitBlog/
cp -rn ./astro-temp/ ./GitBlog/

# 3. 删除临时文件夹
rm -rf ./astro-temp
```

### Step 4：适配 Front Matter & 搬回文章
Astro 默认的文章路径在 `src/content/blog/`。

1.  **搬回文章**：
    ```bash
    mkdir -p GitBlog/src/content/blog/
    mv ../temp_posts/* GitBlog/src/content/blog/
    ```
2.  **适配 Config (zod schema)**：
    Hexo 的 `date` 格式和自定义字段 (如 `tags`，`categories`，`updated`，`abbrlink`) 会让 Astro 报错。你需要修改 `src/content/config.ts` 来兼容它们：
    ```typescript
    const blog = defineCollection({
      type: 'content',
      schema: z.object({
        title: z.string(),
        description: z.string().optional(),
        // 关键：将 Hexo 的日期字符串强制转换为 Date 对象
        pubDate: z.coerce.date(), 
        updatedDate: z.coerce.date().optional(),
        heroImage: z.string().optional(),
        // 添加 Hexo 自定义字段
        tags: z.array(z.string()).optional(),
        categories: z.array(z.string()).optional(),
        abbrlink: z.string().optional(),
      }),
    });
    ```

### Step 5：本地预览与验证
```bash
cd GitBlog
npm install
npm run dev
```
访问 `http://localhost:4321/`，你会看到一个全新的、速度极快的 Astro 博客，而且你的旧文章都在！

---

## 云端 CI/CD 换脑：GitHub Actions 适配

由于引擎更换，原本的 `deploy.yml` 需要彻底重写。Astro 的编译产物在 `dist/`，我们需要用 `npm run build` 替换 `hexo g`。

全新的 `.github/workflows/deploy.yml`：

```yaml
name: Astro Auto Deploy

on:
  push:
    branches:
      - main

env:
  FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source
        uses: actions/checkout@v4
        with:
          submodules: true

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '24'
          cache: 'npm'

      - name: Install & Build
        run: |
          npm install
          npm run build

      - name: Configure SSH
        env:
          SSH_ID_RSA: ${{ secrets.HEXO_DEPLOY_PRI }}
        run: |
          mkdir -p ~/.ssh/
          echo "$SSH_ID_RSA" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts

      - name: Deploy to Private Static Repo
        run: |
          # 进入 Astro 的构建产物目录
          cd dist
          # 初始化临时 Git 环境并推送到静态私有仓库
          git init
          git config --global user.name "your_username"
          git config --global user.email "your_email@example.com"
          git remote add origin git@github.com:yourname/yourname.github.io.git
          git checkout -b main
          git add .
          git commit -m "Rendered by Astro"
          # 强制覆盖，确保结构纯净
          git push -f origin main
```

> **注意**：Cloudflare Pages 无需做任何修改。它只管监听静态仓库 `yourname.github.io`，无论是 Hexo 推送的 `public` 还是 Astro 推送的 `dist`，对于 Cloudflare 来说，它只看 `.html` 文件。

---

## 扫尾工作：仓库改名与 Obsidian 优化

为了更清晰地表达，我将远程源码仓库改名为 `static-blog-source`。

### 5.1 本地与 Submodule 同步
```bash
# 1. 更新本地 Git 远程地址
cd GitBlog
git remote set-url origin git@github.com:yourname/static-blog-source.git

# 2. 回到父目录 (Obsidian 主库)，同步子模块路径
cd ..
# 慎重：确保你已经在 GitHub 网页端完成了改名操作
git submodule sync
```

### 5.2 Obsidian 侧边栏正则过滤 (File Explorer++)
由于 Astro 的目录结构复杂，我们需要更新正则过滤，让侧边栏只显示文章目录：
**Hide Filter：**
```regex
^GitBlog/(?!(src/|src$)).*|^GitBlog/src/(?!(content/|content$)).*|^GitBlog/src/content/(?!(blog/|blog$)).*
```

---

## 避坑

1. Hexo 迁移过来 front-matter 内要用 category 而不是 categories！
2. 还有发布日期等，最好全部从模板中复制到 templater
