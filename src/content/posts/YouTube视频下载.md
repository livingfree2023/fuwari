---
title: YouTube视频下载
category: 技术
tags:
  - Youtube
  - Aria2
  - VPS
published: 2026-05-21T13:05:81+08:00
image: https://image.heavenroad.org/Pasted%20image%2020260524094607.png
slug: slug20260521130581
upload: false
Last Modified: 2026-07-15 11:07:19
---

[原作者：快乐的出帆](https://www.nodeseek.com/post-740487-1)

Rt，昨晚摸索出的在 VPS 上下载 YouTube 8K 高码率视频的命令行工具。`yt-dlp + aria2` 已在 Zouter、DMIT、BWH 的 VPS 成功验证。`yt-dlp` 和 `aria2` 均为开源命令行下载工具，开源地址：

- `yt-dlp`：[link text](https://www.nodeseek.com/jump?to=https%3A%2F%2Fgithub.com%2Fyt-dlp%2Fyt-dlp)
- `aria2`：[link text](https://www.nodeseek.com/jump?to=https%3A%2F%2Fgithub.com%2Faria2%2Faria2)

| VPS | yt-dlp 单线程平均下载速率 | yt-dlp 10 线程平均下载速率 | yt-dlp + aria2 16 线程平均下载速率 |
| --- | --- | --- | --- |
| Zouter HK | 34 MB/s | 74.43 MB/s | 183.34 MB/s |
| DMIT CORONA | 39.25 MB/s | 与单线程接近 | 112.60 MB/s |
| BWH MiniBox |  |  | 98 MB/s |

附 Zouter-HK `yt-dlp + aria2` 16 线程平均下载的截图：

![](https://image.heavenroad.org/7VmKgXONzQKOY1UrbfqX5GFOGfNM9Y9D.webp)

**测速说明**：由于 8K 视频体积太大，转码也会占用部分硬盘空间，BWG 的 MiniBox VPS 硬盘空间限制，所以有些测试未能完成。另由于 YouTube 反扒机制，上述数据也未做多次测试取平均值，仅供参考。

---

## Debian 系统保姆级安装说明书

其他系统可参考，用 AI 修改命令运行。

以下命令均在 root 用户下运行。

### 1. 安装 curl、ffmpeg、aria2
如果已安装可忽略。

```bash
apt update && apt install curl ffmpeg aria2 -y
```

验证安装：

```bash
ffmpeg -version && curl --version
```

下载 `yt-dlp`：

```bash
curl -L https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp -o /usr/local/bin/yt-dlp
```

赋予执行权限：

```bash
chmod a+rx /usr/local/bin/yt-dlp
```

验证安装：

```bash
yt-dlp --version
```

如果输出类似 `2026.03.17` 的版本号，则表示安装成功。

### 2. 安装并配置 JavaScript 运行环境

```bash
apt update && apt install -y nodejs
```

验证安装：

```bash
node -v
```

如果输出类似 `v18.20.4` 的版本号，则表示安装成功。

写入配置文件，让 `yt-dlp` 永久识别 Node.js：

```bash
mkdir -p ~/.config/yt-dlp
echo '--js-runtimes node:/usr/bin/node' > ~/.config/yt-dlp/config
```

### 3. 配置 Cookies
如果未遇到风控，可不设置 Cookies；多线程下载建议设置。

- 登录账号：在你的常用浏览器（如 Chrome 或 Firefox）中，确保你已经成功登录 Google/YouTube 账号。
- 导出 Cookies：使用浏览器扩展（如 Get cookies.txt LOCALLY），下载网址：[link text](https://www.nodeseek.com/jump?to=https%3A%2F%2Fchromewebstore.google.com%2Fdetail%2Fget-cookiestxt-locally%2Fcclelndahbckbenkjhflpdbgdldlbecc%25EF%25BC%2589)
- ![](https://image.heavenroad.org/ojBSiz82RXop5EQfqwMFtqDUpBa0ruZA.webp)
- ![](https://image.heavenroad.org/kOdI0MUGJxd9UFlToTCtsHkfH6kasWTl.webp)
- 下载 `cookies.txt` 文件到本地电脑，并上传到 VPS 的 `/media` 文件夹内。放置位置可自选，但必须与下载命令中的路径保持一致。
- ![](https://image.heavenroad.org/PhW8hP4r67T1vpszxYWAk0LQYA8rY7S4.webp)

### 4. 正式下载

`yt-dlp` 单线程下载命令：

```bash
yt-dlp --js-runtimes node --cookies /media/cookies.txt -f bestvideo+bestaudio -P "/media" "https://www.youtube.com/watch?v=rt1htKl27wE"
```

![](https://image.heavenroad.org/MSEaYqirDGQMxZr76KNO1mcadXdstJw5.webp)

`yt-dlp` 10 线程下载命令：

```bash
yt-dlp --js-runtimes node --cookies /media/cookies.txt -f bestvideo+bestaudio -P "/media" --concurrent-fragments 10 "https://www.youtube.com/watch?v=rt1htKl27wE"
```

![](https://image.heavenroad.org/D2kZnv6wfiAK09zHluUgKWWUGgXuNUwL.webp)

数字 `10` 可以根据实际效果调整，一般 `8` 到 `16` 是最常用的范围，太高反而可能被 YouTube 限速。

![](https://image.heavenroad.org/wN8GScwEQD1YlePlY4oDvdxjpaiweMvI.webp)
