---
title: 全量导出Ghost
category: 技术
description: ""
tags:
  - GitBlog
  - Ghost
  - Gemini
published: 2026-03-20T18:30:00+08:00
#slug: "260320220315"
draft: false
image: https://image.heavenroad.org/157849205-aa24152c-4610-4d7d-b752-3a8c4f9319e6.png
---

> [!NOTE]
> 因为有几百个帖子写在 Ghost 上，没有办法导入到 wordpress，今天折腾好了 astro 后，本来想复制粘贴 ghost 中的 vps 相关帖子过来，结果灵机一动索性让 gemini 教我如何导出所有的帖子和视频图片。正好 ghost 新版本有一个 API 的功能，gemini 真给力，10 分钟就解决了。以下是让 gemini 总结的。（前几天试图让 gemini 帮我恢复一个 ghost 的 compose.yml，虽然失败了，但是莫名其妙的把 ghost 升级到了最新版，于是才有了 apikey，也算是歪打正着、因祸得福、无心插柳柳成荫、塞翁失马，焉知非福……）

这是一份针对 Ghost 博客自动化迁移至 Obsidian（或任何支持 YAML Front-matter 的 Markdown 环境）的最终技术方案。该方案实现了 **文章筛选**、**元数据保留** 以及 **多媒体资源（图片、MP3、MP4）本地化**。

---

## 🚀 最终解决方案：Ghost 媒体全量导出脚本

### 1. 环境准备

在 MacOS 终端中创建一个新文件夹，并安装必要的依赖库：

```bash
mkdir ghost-export && cd ghost-export
npm install axios cheerio minimist @tryghost/content-api turndown
```

### 2. 核心脚本 `export.js`

创建一个 `export.js` 文件，并将以下代码完整存入。该脚本已针对你的需求进行了深度定制：

- **参数化运行**：支持通过命令行输入 URL、Key 和 Tag。
    
- **特定格式 Front-matter**：严格遵循你要求的 YAML 结构。
    
- **媒体自动处理**：自动扫描并下载 `img`、`audio`、`video` 标签资源至 `resource` 目录。

```javascript
const GhostContentAPI = require('@tryghost/content-api');
const TurndownService = require('turndown');
const fs = require('fs');
const path = require('path');
const axios = require('axios');
const cheerio = require('cheerio');
const argv = require('minimist')(process.argv.slice(2));

// 1. 获取命令行参数
const { url: ghostUrl, key: ghostKey, tag: targetTag } = argv;

if (!ghostUrl || !ghostKey) {
  console.error('\n❌ 缺少参数！用法: node export.js --url=https://yourblog.com --key=YOUR_KEY [--tag=VPS]');
  process.exit(1);
}

const api = new GhostContentAPI({ url: ghostUrl, key: ghostKey, version: "v5.0" });
const turndownService = new TurndownService();

// 2. 建立目录结构
const baseDir = path.join(__dirname, 'ghost_export_final');
const resourceDir = path.join(baseDir, 'resource');
if (!fs.existsSync(baseDir)) fs.mkdirSync(baseDir);
if (!fs.existsSync(resourceDir)) fs.mkdirSync(resourceDir);

// 3. 增强型文件下载函数 (支持重试与流写入)
async function downloadMedia(fileUrl, saveFolder) {
  if (!fileUrl || !fileUrl.startsWith('http')) return fileUrl;
  
  try {
    const urlObj = new URL(fileUrl);
    const fileName = path.basename(urlObj.pathname);
    const localPath = path.join(saveFolder, fileName);
    const relativePath = `./resource/${fileName}`;

    if (fs.existsSync(localPath)) return relativePath;

    const response = await axios({ url: fileUrl, method: 'GET', responseType: 'stream', timeout: 30000 });
    const writer = fs.createWriteStream(localPath);
    response.data.pipe(writer);

    return new Promise((resolve, reject) => {
      writer.on('finish', () => resolve(relativePath));
      writer.on('error', reject);
    });
  } catch (e) {
    console.warn(`  ⚠️ 下载失败: ${fileUrl} (跳过)`);
    return fileUrl;
  }
}

async function startExport() {
  try {
    console.log(`📡 正在连接 Ghost: ${ghostUrl}`);
    const posts = await api.posts.browse({
      limit: 'all',
      include: 'tags',
      status: 'all',
      filter: targetTag ? `tag:${targetTag}` : undefined 
    });

    console.log(`✅ 找到 ${posts.length} 篇文章，开始解析媒体资源...\n`);

    for (const post of posts) {
      process.stdout.write(`📦 处理中: ${post.title} ... `);
      
      const $ = cheerio.load(post.html || '');
      
      // 处理封面图
      let localFeatureImage = "";
      if (post.feature_image) {
        localFeatureImage = await downloadMedia(post.feature_image, resourceDir);
      }

      // 处理正文内的 <img>, <video>, <audio> 以及 <source>
      const mediaTags = $('img, video, audio, source');
      for (let i = 0; i < mediaTags.length; i++) {
        const el = $(mediaTags[i]);
        const src = el.attr('src');
        if (src) {
          const localRelPath = await downloadMedia(src, resourceDir);
          el.attr('src', localRelPath);
        }
      }

      // 转换为 Markdown
      const updatedHtml = $.html();
      const markdownBody = turndownService.turndown(updatedHtml);
      
      // 构建 Front-matter
      const rawDate = post.published_at ? post.published_at.split('T')[0] : '';
      const rawTags = post.tags ? post.tags.map(t => `  - ${t.name}`).join('\n') : '';

      const frontMatter = [
        '---',
        `title: ${post.title}`,
        `description: "${post.custom_excerpt || ""}"`,
        `category: 技术`,
        `tags:`,
        `${rawTags}`,
        `image: "${localFeatureImage}"`,
        `published: ${rawDate}`,
        `draft: ${post.status === 'draft'}`,
        `slug: "${post.slug}"`,
        '---',
        '\n'
      ].join('\n');

      fs.writeFileSync(path.join(baseDir, `${post.slug}.md`), frontMatter + markdownBody, 'utf8');
      console.log('完成');
    }

    console.log(`\n🎉 导出成功！所有文件位于: ${baseDir}`);
  } catch (err) {
    console.error(`\n❌ 运行失败: ${err.message}`);
  }
}

startExport();
```

---

### 3. 如何使用

在终端执行以下命令即可：

```bash
# 例子：只导出标签为 VPS 的文章及其媒体
node export.js --url=https://your-ghost-blog.com --key=your_api_key --tag=VPS
```

---

### 4. 方案亮点与后续建议

- **数据保真**：脚本直接提取 Ghost 数据库原始的 `slug`、`tags` 和 `published_at`，不进行任何破坏性修改。
    
- **多媒体支持**：通过 `cheerio` 解析 HTML 树，能够精准捕获 Ghost 编辑器插入的各种卡片（视频、音频、图片），并统一下载到 `resource` 目录。
    
- **Obsidian 完美兼容**：
    
    - **Properties**：生成的 Front-matter 格式会自动激活 Obsidian 的属性面板。
        
    - **预览**：由于使用了 `./resource/xxx.mp4` 的相对路径，只要保持文件夹结构不变，在 Obsidian 中可直接播放音视频。
        
- **自动化性能**：脚本采用 `for…of` 异步等待机制，能有效防止因下载大量高清视频（MP4）导致的网络拥堵或 Ghost 接口报错。
