---
title: Obsidian自动化构建博客6-插入图片
description: ""
category: 技术
tags:
  - GitBlog
published: 2026-03-20T13:27:00+08:00
draft: false
#slug: "20260320132700"
image: https://image.heavenroad.org/resources/Pasted%20image%2020260322205858.png
---

## Obsidian 中直接插入图片

### Take 1
先用 ob 内配置 *Files and link* - *Default Location* - *Subfolder* 试试看
直接复制粘贴

在 posts 下新增了一个 resources 目录图片名称包含空格，现在 push 试试看

…….

直接就成功了，真简单~
但是这个图片每次编译都会变链接，所以浏览器无法缓存，所以要上个难度，把图片上传 R2，确保 url 固定。

先试试插件 *Image Upload Toolkit*
奇怪，没有自动上传
执行 *Publish Image*
看到 R2 里面有文件了，但是连接没更新啊，而且我本地 resources 还是增加了这个文件，怎么回事？

---

换一个 plugin *Custom Image Auto Uploader*，需要 picgo-core, 在香港小鸡部署后配置 R2 然后运行 `picgo server --secret PASSWORD -h 0.0.0.0`

再试试，还是不行……

注册了七牛再试试……

七牛太恶心，实名注册还要在他那里买域名否则只有 30 天的临时域名，放弃

七牛这个域名设置让我想再试试 *Image Upload Toolkit* ，加 custom domain 看看……

![](https://image.heavenroad.org/Gemini_Generated_Image_13hnrg13hnrg13hn.png)

啊成功了，这个插件的逻辑是全部写完 markdown 以后通过 publish command 执行，并非粘贴的时候直接执行。而且如果 markdown 中有任意一张图片引用无法找到对象，插件就会停止运行，导致最后失败。刚才前面 5 张图片都成功，最后一张不成功（因为 markdown 中引用了一个不存在的图片，没想到刚才是因为这个）。

但这个插件的问题是，本地 vault 里面还有一个附件，要手动清理……

搞了个插件 obsidian://show-plugin?id=file-cleaner-redux 用它清楚多余的文件，先手动配置包含的文件 `.jpg, .png, .gif`
试试看

成功！
