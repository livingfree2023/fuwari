---
title: Obsidian自动化构建博客总结
category: 技术
tags:
  - GitBlog
published: 2026-03-23T11:03:41+08:00
draft: false
image: https://image.heavenroad.org/resources/Pasted%20image%2020260322205756.png
---

```mermaid
graph

	Start[Obsidian]
	GitPlugin[Git插件]
	ShellPlugin[shell command 插件]
	subgraph Path1[无 VPS]
		P1_ObidianGithub[GitHub Obsidian（私库）]
		P1_Fuwari[Fuwari的fork（公库）]
		P1_ObidianGithubAction[用Github Action</br> 监控blog目录并推送]
		P1_Pages[GitHub Pages 编译部署<br/>xxx.github.io <br>DNS污染]
		P1_Vercel[Vercel 编译部署<br/>xxx.vercel.app <br>DNS污染]
		P1_CloudflarePage[Cloudflare Pages </br>编译部署</br>xxx.pages.dev <br>DNS污染]
	end

	subgraph Path2[有 VPS]
		P2_VPSGit[仓库在 VPS]
		P2_VPSGitBuild[Post-Receive Hook</br> 触发 Build]
		P2_Github[仓库在 Github]
		P2_WatchGithub[contab 定时触发</br> Pull + Build]
		P2_Fuwari[Fuwari Clone</br> VPS 编译部署]
		P2_Caddy[Caddy/Nginx]
		P2_LocalEnv[本地建 Fuwari Clone</br>软链接 blog 目录]

		subgraph Manual[手动触发]
			P2_Localbuild[手动触发本地编译]
			P2_SSHbuild[手动触发推送<br>远程编译]
			P2_Rsync[rsync 增量同步]
		end
	end

	User[End User Browser]

	Start --> GitPlugin
	Start --> ShellPlugin
	GitPlugin -- 方案1 --> P1_ObidianGithub --> P1_ObidianGithubAction --> P1_Fuwari --> P1_Pages --> User
	P1_Fuwari --> P1_CloudflarePage --> User
	P1_Fuwari --> P1_Vercel --> User
	GitPlugin -- 方案2 --> P2_VPSGit --> P2_VPSGitBuild --> P2_Fuwari --> P2_Caddy --> User
	GitPlugin -- 方案3 --> P2_Github --> P2_WatchGithub --> P2_Fuwari
	ShellPlugin -- 方案5 --> P2_LocalEnv --> P2_Localbuild --> P2_Rsync --> P2_Caddy
	ShellPlugin -- 方案4 --> P2_SSHbuild --> P2_Fuwari
```

## 前提

Obidian 作为主要笔记仓库，其中建一个目录，里面只存放博客内容。可以通过部署 hexo/astro 等框架编译成静态 html。
 1. 实现和框架解耦，换框架不影响博客文章。
 2. 私库内容完全私有，只有博客目录的内容会被传输到仓库外。
 3. 实现全自动备份，因为文章在 obsidian vault 中，本来也要备份 vault，自动就一起备份了。
 4. 实现全自动发布，写作完成后内容自动编译发布上线。
 
## 方案解析

|      | 全自动 | 配置难度     | 发布速度     | 无须开发环境 | 支持手机发布 | 可直连 |
| ---- | --- | -------- | -------- | ------ | ------ | --- |
| 方案 1 | ✅   | ⭐️       | ⭐️       | ✅      | ✅      | ❌   |
| 方案 2 | ✅   | ⭐️⭐️     | ⭐️⭐️⭐️   | ✅      | ✅      | ❌   |
| 方案 3 | ✅   | ⭐️⭐️     | ⭐️⭐️⭐️   | ✅      | ✅      | ❌   |
| 方案 4 | ❌   | ⭐️⭐️⭐️   | ⭐️⭐️⭐️   | ✅      | ❌      | ✅   |
| 方案 5 | ❌   | ⭐️⭐️⭐️⭐️ | ⭐️⭐️⭐️⭐️ | ❌      | ❌      | ✅   |

我一共折腾了 8 种部署方式，全部都能跑通，最后筛选觉得这 5 种还比较不错。（目前自己用方案 2）
白嫖 CF R2 做图床，插件可自动替换 URL

## 在 Obsidian 中写作流程

1. 编写 markdown，图片可以直接粘贴进来，图片会暂存在自文件夹的 resource 目录，不用管它，不用上传（可以考虑关闭 git 自动 commit）。
2. CMD+P 执行 `Image Upload Toolkit: Publish Page` 会自动把图片上传 R2，同时更新文章内的链接，但是不会自动删除本地文件。
3. CMD+P 执行 `File Cleaner Dux: Clean files` , 因为 resource 下粘贴的文件不再被任何文件引用，用这个插件自动清理 vault 里的所有图片，节约空间。

## 关键设置
![](https://image.heavenroad.org/Pasted%20image%2020260321230001.png)

这个设置影响所有的图片路径，图片上传 R2 之后会在 bucket 里面自动建一个 resource 目录，未来所有的链接都会包含 resource，不要轻易改动。

## Templater 插件，推荐但非必要

- 建一个目录放模板
- 新建一个模板笔记：`Blog-Front-Matter`:
```
---
title: Post Front-matter
category:
description: ""
tags:
  - 标签
image: "" 如果有封面图就不能加引号，引号为了避免编译错误
published: 2026-03-23T13:03:82+08:00 
draft: false
---
```
- 配置一个 **folder template**, 指定 blog 目录对应的 template 为 `Blog-Front-Matter`

## Git 插件
![](https://image.heavenroad.org/Pasted%20image%2020260321222728.png)
![](https://image.heavenroad.org/Pasted%20image%2020260321222807.png)
![](https://image.heavenroad.org/Pasted%20image%2020260321222918.png)

## File Cleaner Redux 插件
![](https://image.heavenroad.org/Pasted%20image%2020260321222954.png)
![](https://image.heavenroad.org/Pasted%20image%2020260321223014.png)

## Image Upload Toolkit 插件（第三方图床才需要）
![](https://image.heavenroad.org/Pasted%20image%2020260321223104.png)
![](https://image.heavenroad.org/Pasted%20image%2020260321223122.png)

### Cloudflare R2 bucket 文档说明
- Enable public access
- Expose your bucket as a custom domain under your control.
- Expose your bucket using a Cloudflare-managed `https://r2.dev` subdomain for non-production use cases.

## Linter 插件，不必须但强烈推荐，优化中英文混排 markdown
- Quote 这个开关和 templater 里面的模板冲突，必须关掉
![](https://image.heavenroad.org/Pasted%20image%2020260321223329.png)
- 这个必须开启，否则中英文混合的 markdown 排版可能出错
![](https://image.heavenroad.org/Pasted%20image%2020260321223534.png)

## Shell Command 插件，如果要本地操作上传或编译

- 编译和同步的脚本，前提是 VPS_HOST 已经配置好 `ssh public key` 和 `.ssh/config`
```bash
source ~/.zshrc && \
# 1. 进入工厂目录 lint 一下内容
cd ~/static-blog-source && \
# 2. 彻底清理旧产物并重新编译
pnpm run build && \
# 3. 只有在编译成功(dist/index.html 存在)时才同步到 VPS
[ -f "./dist/index.html" ] && \
rsync -avz --delete ./dist/ VPS_HOST:/var/www/html/blog/

```
- Output - stdout - ignore
- Output - stderr - ask after execution （可以方便的看结果，熟练后无所谓）

## One more thing
Obsidian 处理 markdown 链接和 astro 处理链接还是有区别
比如从 `aaa.md` 添加 `bbb.md` 的链接，wikilink 写法是 `[[bbb.md]]`, markdown 的写法是 `[bbb](bbb.md)` 但是 `bbb.md` 这个笔记被 astro 编译后地址变成 `domain.com/posts/bbb/`, 这个链接指向被编译成 `domain.com/posts/aaa/bbb.md/` ，如果要保证编译后的地址正确，在 obsidian 中这个链接就没法正确跳转。两者不可兼得。
这个问题应该不止 fuwari 有，所有 astro 都不会自动转化。
有一个办法是使用 `static site MD exporter` 插件，并且为每个 post 指定一个 slug，插件可以扫描文章并且自动转化链接，然后直接 push 到一个指定的 git 仓库中。但是这个插件需要额外 1. 添加一个专用的 frontmatter 标记；2. 手动触发。Templater 可以直接把这两个 frontmatter 写到模板中，但手动触发现在没有好的办法。我评估 posts 内跳转的情况不多，如果需要的话，只好手动了。
如果要加 slug 的话，templater 里面要特别注意必须是一个 text，用时间戳的话要加文字比较安全，加引号容易被自动删掉，然后就出错了。测试下来用 `slug<% tp.date.now("YYYYMMDDHHMMSS") %>` 是 ok 的。
