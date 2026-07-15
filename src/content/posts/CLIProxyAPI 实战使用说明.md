---
title: CLIProxyAPI 实战使用说明
category: 技术
tags:
  - AI
  - CPA
  - CLIProxyAPI
published: 2026-04-13T19:04:25+08:00
image: https://image.heavenroad.org/Pasted%20image%2020260413195126.png
slug: slug20260413190425
upload: false
Last Modified: 2026-07-15 11:07:18
---

## 1. 配置详细解说

这篇文章是对 [CLIProxyAPI项目](https://github.com/router-for-me/CLIProxyAPI) 配置文件中各配置项的详细解读，供程序使用者有疑问时参阅

温馨提示：配置文件支持热重载，修改配置文件是即时生效的，不需要重启程序。

```
# 端口号，CLIProxyAPI运行了个HTTP服务器，需要端口号来进行访问
port: 8317

# 远程管理配置，配合EasyCLI或者WebUI来使用
remote-management:
  # 启用远程管理的开关，如果你部署在服务器上
  # 那么需要设置为true，才能使用EasyCLI或者WebUI连接到CLIProxyAPI进行管理
  # 如果只是本地使用API进行管理的，可以保持false不动
  allow-remote: false

  # 如果想使用EasyCLI或者WebUI通过API对CLIProxyAPI进行管理，必须设置Key
  # 如果不设置，视同关闭了API管理功能，就无法使用EasyCLI或者WebUI进行连接了
  # 如果你不需要使用EasyCLI或者WebUI进行管理，可以留空
  secret-key: ""

# 认证文件存放目录，用于存放Gemini CLI、Gemini Web、Qwen Code、Codex的认证文件
# 默认设置，是在你当前账户目录下的.cli-proxy-api文件夹，适配Windows和Linux环境
# 程序首次启动时会自动创建该文件夹
# Windows下默认为C:\Users\你的用户名\.cli-proxy-api
# Linux下默认为/home/你的用户名/.cli-proxy-api
# 如果在Windows环境下使用非默认位置，需要参照这样的格式修改填写"Z:\\CLIProxyAPI\\auths"
auth-dir: "~/.cli-proxy-api"

# 是否在日志中启用Debug信息，默认不启用，需要作者配合排错的时候打开就行
debug: false

# 是否将日志重定向到日志文件中
# 默认启用，日志会保存在程序目录下的logs文件夹中
# 如果关闭的话，会在控制台显示日志
logging-to-file: true

# 开关使用统计，默认启用
# 需要使用API来查看使用量，可以用EeasyCLI或者WebUI来查看
usage-statistics-enabled: true

# 如果你要使用代理，那么需要进行以下的设置，支持socks5/http/https协议
# 按照这样的格式"socks5://user:pass@192.168.1.1:1080/"填写
proxy-url: ""

# 当请求碰到403, 408, 500, 502, 503, 504这些错误码的时候，程序自动重试请求的次数
request-retry: 3

# 模型受到限制之后的处理行为
quota-exceeded:
  # 多账号轮询的核心配置
  # 设置为true时，例如一个账号触发了429，程序会自动切换到下一个账号重新发起请求
  # 设置为false时，程序会把429的错误信息发给客户端，结束当前请求
  # 也就是说，当设置为true时，只要轮询的账号里至少有一个号是正常的，客户端这里就不会报错
  # 而设置false时，则需要客户端来进行重试或中止操作
  switch-project: true 
  # Gemini CLI独占配置，适用于Gemini 2.5 Pro和Gemini 2.5 Flash模型
  # 当正式版配额用完之后，会自动切换到Preview模型，保持开启即可
  switch-preview-model: true

# 各种AI客户端访问CLIProxyAPI所需要填写的Key，就在这里设置，和后边的各种Key不要弄混淆了
# 通俗点讲，这里的Key是CLIProxyAPI作为服务器所需要设置的
# 后边的各种Key是CLIProxyAPI作为客户端去访问服务器所需要的
api-keys:
  - "your-api-key-1"
  - "your-api-key-2"

# Gemini的官方API Key，如果你已经配了Gemini CLI，那么不建议填
# 因为Gemini CLI是满血的，而官方Key是残血的，填了的话会一起参与轮询
generative-language-api-key:
  - "AIzaSy...01"
  - "AIzaSy...02"
  - "AIzaSy...03"
  - "AIzaSy...04"

# Codex的API Key，各种中转站提供的Codex的key和base-url参数，填在这里就可以接入了
codex-api-key:
  - api-key: "sk-atSM..."
    base-url: "https://www.example.com"

# Claude的API Key，使用官方Key的时候，不要填base-url，使用第三方中转的，填base-url
claude-api-key:
  - api-key: "sk-atSM..."
  - api-key: "sk-atSM..."
    base-url: "https://www.example.com"

# 各种OpenAI兼容的都可以在这里接入，不多解释了
openai-compatibility:
  - name: "openrouter"
    base-url: "https://openrouter.ai/api/v1"
    api-keys:
      - "sk-or-v1-...b780"
      - "sk-or-v1-...b781"
    models:
    	# OpenAI兼容供应商提供的模型名称
      - name: "moonshotai/kimi-k2:free"
      	# 模型别名
        alias: "kimi-k2"

# Gemini Web的相关设置，可以忽略掉，用默认值就行
gemini-web:
    # 此选项用于状态化会话，由于Gemini Web是逆向的
    # 因而如果设置false的话，每条发送的消息程序会携带之前的所有上下文发送给服务器
    # 设置true的话，程序会按客户端发送的报文，根据最长匹配寻找之前的会话
    # 如果已有会话，则只发送当前的消息，而不携带所有上下文
    # 如果使用Nano Banana模型，请务必保持此选项为true，否则无法进行连续会话修图
    # 如果还有不理解的，可以开始切换开关后，在Gemini Web官方网页查看效果
    context: true
    # 最大发送的字符，保持为默认值即可
    max-chars-per-request: 1000000
    # 程序默认超出最大字符，会进行截断，分批发送，截断时，会在报文最后附加一条让模型等待的消息
    # 如果设置true，则不会在报文最后附加这条消息
    # 建议保持false即可，因为只有截断才会附加消息，非截断情况是不会附加的
    disable-continuation-hint: false
    # 编程模式，不用来进行编程，请不要启用，使用Nano Banana模型，请务必关闭
    # 设置ture，会有以下效果
    ## 使用系统自带的编码助手Gem进行对话
    ## 对话时如有思考内容，会把思考内容并入正文
    ## 在报文最后附加一条关于XML的消息
    code-mode: false
```

---

## 2. 项目介绍 +Qwen 实战

CLIProxyAPI 是一款使用 Go 语言编写的开源 AI 代理工具。也许是因为名字朴实无华，许多人可能对它还很陌生。在遇到它之前，为了能“白嫖” Gemini 模型，我曾先后折腾过 AIStudioProxyAPI、AIStudio-Build-Proxy、Gemini-FastAPI 等多款反代工具，但它们或多或少都有些不尽如人意的地方。

直到我发现了 CLIProxyAPI 并深度使用了几个月，我可以肯定地说：无论是在性能、功能还是适用性上，它都是我用过最出色的 AI 代理工具，没有之一。称之为“神器”也毫不为过。

**官方仓库地址**：[https://github.com/router-for-me/CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI)

### **它究竟能做什么？**

| 功能特性                                                 | 支持模型                       |
| -------------------------------------------------------- | ------------------------------ |
| 为 CLIm 模型提供 OpenAI/Gemini/Claude/Codex 兼容的 API 端点    | gemini-2.5-pro                 |
| 新增 OpenAICodex（GPT 系列）支持（OAuth 登录）              | gemini-2.5-flash               |
| 新增 ClaudeCode 支持（OAuth 登录）                          | gemini-2.5-flash-lite          |
| 新增 QwenCode 支持（OAuth 登录）                            | gemini-2.5-flash-image-preview |
| 新增 GeminiWebHost 支持（通过 Cookielogin）                 | gpt-5                          |
| 支持流式与非流式响应                                     | gpt-5-codex                    |
| 函数调用/工具支持                                        | claude-opus-4-1-20250805       |
| 多模态输入（文本、图片）                                 | claude-opus-4-20250514         |
| 多账户支持与轮询负载均衡（Gemini、OpenAI、Claude 与 Qwen） | claude-sonnet-4-5-20250929     |
| 简单的 CLI 身份验证流程（Gemini、OpenAI、Claude 与 Qwen）    | claude-sonnet-4-20250514       |
| 支持 GeminiAIStudioAPI 密钥                                | claude-3-7-sonnet-20250219     |
| 支持 GeminiCLIm 多账户轮询                                 | claude-3-5-haiku-20241022      |
| 支持 ClaudeCode 多账户轮询                                 | qwen3-coder-plus               |
| 支持 QwenCode 多账户轮询                                   | qwen3-coder-flash              |
| 支持 OpenAICodex 多账户轮询                                |                                |
| 通过配置接入上游 OpenAI 兼容提供商（例如 OpenRouter）       |                                |
| 可复用的 GoSDK                                            |                                |

简单来说，CLIProxyAPI 的核心优势包括：

-   **无需安装 Gemini CLI**，即可将其授权转换为通用的 API Key，从而在任何应用中调用功能完整的 Gemini 2.5 Pro、Gemini 2.5 Flash、Gemini 2.5 Flash Lite 模型。当正式版模型配额用尽后，它会自动切换到 Preview 模型（如 `gemini-2.5-pro-preview-05-06`），基本能用足每天 1000 次的调用配额，轻松实现“Gemini 自由”。

-   **无需安装 Qwen Code**，即可将其授权转换为通用的 API Key，在任何地方调用 Qwen3 Coder Plus、Qwen3 Coder Flash 模型，实现“Qwen3 Coder 自由”。

-   **无需安装 Codex**，即可将其授权转换为通用的 API Key，在任何地方调用 GPT-5、GPT-5-Codex 模型。尤其在目前可以免费开设 Team 账户的活动下，轻松实现“GPT 自由”。

-   **将 Gemini 网页版转换为 API Key**，在任何地方调用 Nano Banana 等网页版模型（需客户端支持。据网友分享，免费版 Gemini 网页账户每天可调用约 100 次，而 Gemini Pro 用户则高达 1000 次）。

-   **强大的负载均衡能力**。CLIProxyAPI 支持将不同来源（无论是 API Key 还是 OAuth 授权）的多个账户整合在一起进行负载均衡轮询，这意味着你可以轻松地将调用配额翻倍。

-   **极低的资源消耗**。值得一提的是，该程序对系统资源的消耗极低。程序本身仅 10MB 左右，启动时内存占用不到 10MB，长时间峰值内存占用也仅有 100MB 左右，几乎任何电脑都能流畅运行。

程序的使用非常简单。官方不仅提供了适用于各平台的二进制文件和 Docker 部署方式，还提供了 EasyCLI 和 WebUI，对新手十分友好。所有设置均通过 `config.yaml` 配置文件管理，且支持热重载——修改配置后即时生效，无需重启程序。

### **实战教程：转换 Qwen Code 为 API Key**

下面，我们以在 Windows 平台下将 Qwen Code 转换为 API Key 为例，演示 CLIProxyAPI 的具体使用方法。

1.  **下载并解压**

    首先，从官方仓库下载预编译的可执行文件，并将其解压到任意文件夹。在本例中，我将其放在 `Z:\CLIProxyAPI` 目录下。我们只需要用到图中的两个文件。

    ![](https://image.heavenroad.org/b247eb98e0172c799c452d647ab09836.png)

2.  **编辑配置文件**

    将 `config.example.yaml` 重命名为 `config.yaml`，然后用文本编辑器打开，仅需保留并修改以下基础配置项：

    ```yaml
    port: 8317
    
    # 文件夹位置请根据你的实际情况填写
    auth-dir: "Z:\\CLIProxyAPI\\auths"
    
    request-retry: 3
    
    quota-exceeded:
      switch-project: true
      switch-preview-model: true
    
    api-keys:
    # Key请自行设置，用于客户端访问代理
    - "ABC-123456"
    ```

3.  **获取授权**

    在 `CLIProxyAPI` 目录下打开终端，输入 `cli-proxy-api --qwen-login` 后回车。程序会自动打开浏览器，请在浏览器中登录你的 Qwen 账户并完成授权。

    ![](https://image.heavenroad.org/ec2e867f5a8de1d24969cb5225cdaa9e.png)

    完成授权后，回到终端，程序会尝试获取认证信息。成功后，会要求输入邮箱或昵称（如图中红色箭头所示）。这只是一个用于标识账户的别名，可以随意填写。我这里填的是 `qwen-example`。回车后，可以看到认证文件已成功生成，并保存到了配置文件 `auth-dir` 指定的位置。

    ![](https://image.heavenroad.org/8d37d893884910db77bd4498373a4922.png)

    **提示**：如果系统没有自动弹出浏览器，请不必担心。手动复制终端中红框标出的网址，粘贴到浏览器中打开即可完成授权。

4.  **启动代理服务**

    以上步骤完成了账户认证。现在，我们来正式启动代理服务。直接双击可执行文件 (`cli-proxy-api.exe`)，出现以下窗口即代表启动成功。

    ![](https://image.heavenroad.org/9e75a484a7f0713c6b56f1fd9a749195.png)

5.  **在客户端中配置和测试**

    至此，一切准备就绪。下面我们使用 Cherry Studio 来进行测试。

    - 在 Cherry Studio 中添加一个新的模型提供商。

    ![](https://image.heavenroad.org/2a2d401e75d02421861e136a6e377f5b.png)

    - 模型提供商类型可以选择除 Azure 之外的任意类型。这里我们以 `OpenAI-Response` 为例，供应商名称可自定义，例如 `CLIProxyAPI`。

    ![](https://image.heavenroad.org/3161b549ee3877342f178b2828a30dc6.png)

    -   **API 密钥**：填写我们在 `config.yaml` 中自己设置的 Key，本例中为 `ABC-123456`。
    -   **API 地址**：填写我们本地服务的地址和端口。还记得配置文件中的端口号 `8317` 吗？这里我们填入 `http://127.0.0.1:8317`。

    ![](https://image.heavenroad.org/5f9638b5f5e5c14d4a818b7362167083.png)

    - 点击“管理模型”，你就可以看到通过代理加载的 Qwen Code 模型了。

    ![](https://image.heavenroad.org/9a5d68865bfd1d5fa1cf7778860e6a76.png)

    - 添加模型后，我们来测试一下。

    ![](https://image.heavenroad.org/83fd73b257efa9e50ce11bf03cb597d6.png)

可以看到，模型已成功返回消息。整个配置过程是不是很简单？

---

## 2. Codex 和 Gemini CLI

 在之前的文章中，我们通过在 CLIProxyAPI 上的简单配置，成功将 Qwen Code 转换成了 API 并在 Cherry Studio 中调用。相信读到这里的你，已经对这款工具的强大功能和便捷性有了初步认识。

 在本篇教程中，我们将继续探讨，并把 Codex 和 Gemini CLI 也集成进来。

 需要说明的是，本次操作所使用的配置文件与上一篇 Qwen 教程中的是同一个

 ```yaml
 port: 8317
 
 # 文件夹位置请根据你的实际情况填写
 auth-dir: "Z:\\CLIProxyAPI\\auths"
 
 request-retry: 3
 
 quota-exceeded:
   switch-project: true
   switch-preview-model: true
 
 api-keys:
 # Key请自行设置，用于客户端访问代理
 - "ABC-123456"
 ```

### 配置 Codex

 首先，我们来配置 Codex。Codex 的 OAuth 授权流程与之前的 Qwen 非常相似。在终端命令行中输入 `cli-proxy-api --codex-login`，系统会自动打开 ChatGPT 的授权页面，请使用你的 ChatGPT 账号登录。

 ![](https://image.heavenroad.org/8d78b93fcfb3e111a93f6437f9a6acfa.png)

 如果是 ChatGPT Team 账号，则需要选择对应的工作空间。授权成功的页面如下：

 ![](https://image.heavenroad.org/86518a31294a769281c7c5ef8946550e.png)

 回到终端命令行，可以看到认证文件已成功生成并保存。

 ![](https://image.heavenroad.org/c4c9613c9b086a9d8af70d0cf31d75f9.png)

 如果你有多个 ChatGPT 账号，只需重复几次同样的操作即可。

### 配置 Gemini CLI

 接下来，我们来添加 Gemini CLI。Gemini CLI 是完全免费的，但有些用户在配置过程中可能会遇到问题。因此，在这里我将从创建 Google Cloud 项目开始，一步步带你完成整个授权认证过程。

 首先，请用你的 Google 账号登录 https://console.cloud.google.com/ 。登录成功后，点击图中所示位置：

 ![](https://image.heavenroad.org/75416d68babdacc8ddd9bfd652a49b38.png)

 点击“新建项目”。

 ![](https://image.heavenroad.org/566e57546c8bcd73d7ce41e56581b4a5.png)

 给项目命名后，点击“创建”。

 ![](https://image.heavenroad.org/9e90a699265fc1dcfb7755f7c1858b73.png)

 按照第一步的位置，选择刚刚创建的项目。

 ![](https://image.heavenroad.org/a84803f955e057f34723227bdf55b145.png)

 先把红框内的项目 ID 复制下来备用，然后点击左上角箭头所指的位置。

 ![](https://image.heavenroad.org/c1d1617812ecea50c3a63dfb76eb5c04.png)

 依次点击“API 和服务” - “已启用的 API 和服务”。

 ![](https://image.heavenroad.org/782d8096de6276f24fc8ca444ee2910d.png)

 点击“启用 API 和服务”。

 ![](https://image.heavenroad.org/1c0e48d7434bc76d37ef769d86684595.png)

 在图示的搜索框内输入 `cloudaicompanion.googleapis.com`，然后点击搜索到的“Gemini for Google Cloud”。

 ![](https://image.heavenroad.org/a52b17b80635df9e46b8d5b49957e3d6.png)

 点击“启用”。

 ![](https://image.heavenroad.org/58afcf1004138c4c1d63474030d49dff.png)

 至此，Google Cloud 前期的准备工作就全部完成了。现在，我们回到 CLIProxyAPI 程序所在的目录，打开终端命令行，输入 `cli-proxy-api --login --project_id [你的项目ID]`。例如，在本例中就是 `cli-proxy-api --login --project_id mimetic-planet-473413-v7`。

 随后会弹出授权页面，请使用刚才完成准备工作的 Google 账号登录。

 ![](https://image.heavenroad.org/51ca74eb061751336d110b3421c43548.png)

 验证成功的页面如下：

 ![](https://image.heavenroad.org/346f6add9a943f0c324afbad856cc3bf.png)

 回到终端命令行，可以看到认证文件已被成功保存。

 ![](https://image.heavenroad.org/8682dc8a08bffd34d7900819e1073960.png)

 有些读者可能会好奇，为什么 Codex 和 Gemini CLI 在验证成功后的命令行信息，与 Qwen 有所不同？答案是，在验证 Codex 和 Gemini CLI 时，CLIProxyAPI 会在本地监听一个特定端口以接收回调，因此验证总是一次成功。而在验证 Qwen 时，CLIProxyAPI 会直接从 Qwen 的验证服务器来获取授权信息，因此最多会有 60 次的尝试请求。

### 验证模型

 我们再来验证一下刚才通过 OAuth 添加的 Codex 和 Gemini CLI。在 Cherry Studio 中添加模型，如下所示：

 ![](https://image.heavenroad.org/db8e65c9548213303da43cc214ee5000.png)

 试试 Gemini-2.5-Pro：

 ![](https://image.heavenroad.org/a2fc6ce45adcf334a2908984a8db428d.png)

 再来问问 GPT-5-Codex：

 ![](https://image.heavenroad.org/c07a3dbd57f728186ad835fa5afdde6d.png)

 至此，所有模型都已成功集成。你学会了吗？

---

## 3. NanoBanana 实战

经过前两期的实战，我们已成功在 `CLIProxyAPI` 上集成了 `Qwen Code`、`Gemini CLI` 和 `Codex`。本期内容将介绍如何通过添加 Gemini Web 的 Cookie，使 `CLIProxyAPI` 支持 `NanoBanana` 模型。

Gemini 的 `NanoBanana` 模型因其出色的图像处理能力而备受赞誉。然而，Google 并未提供该模型的免费 API。而现在，使用 `CLIProxyAPI` 之后，我们就可以通过集成 Gemini Web，从而以免费 API 的形式使用 `NanoBanana` 啦。

我们有两种方法可以获取认证信息：

### 第一种方法

首先，使用您的 Google 账号登录 [Gemini 官网](https://gemini.google.com/app)。据了解，普通账号每天有 100 次图像生成配额，Pro 账号则有 1000 次。登录成功后，在浏览器中按 F12 打开开发者工具，并切换到“网络” (Network) 选项卡。

![](https://image.heavenroad.org/074fcf1c455e99185ceeada71a27bd8c.png)

在筛选框中输入 `List`，然后将鼠标悬停在您的用户头像上。片刻之后，下方列表中应出现 `ListAccounts` 的条目。如果未出现，请刷新页面重试。

![](https://image.heavenroad.org/7cb7104fa93a6b6a6903e0745d3b5573.png)

点击 `ListAccounts`，在“标头” (Headers) -“请求标头” (Request Headers) 中找到 `Cookie`，并完整复制其值。

![](https://image.heavenroad.org/c2ba085f10fcb145aff7e9d5081b9382.png)

回到 `CLIProxyAPI` 程序所在的目录，打开终端或命令行，输入命令 `cli-proxy-api --gemini-web-auth`。根据提示，粘贴我们刚才复制的 `Cookie` 值并回车，即可看到验证成功的消息，`Cookie` 已被自动保存。

![](https://image.heavenroad.org/e149d07875cb8dab12de95f82d2b3e45.png)

### 第二种方法

如果您使用的是 macOS 系统，或者第一种方法认证失败，那么可能需要手动输入 `__Secure-1PSID` 和 `__Secure-1PSIDTS` 的值。请切换到“应用” (Application) 选项卡，并依次复制图示中的这两个值。

![](https://image.heavenroad.org/e5b5debae5ec74a31a1b527e506895e7.png)

![](https://image.heavenroad.org/7767f178e1186358f1a9a498108e5ac0.png)

在命令行执行验证时，根据提示手动填入这两个值即可完成验证。

![](https://image.heavenroad.org/b02fb7704d5c67385d781f9d9893e0b2.png)

### 验证步骤

接下来我们进行验证。需要注意的是，目前程序仅支持通过 OpenAI 兼容接口和 Gemini 原生接口进行文生图或图文生图的操作。因此，我们之前在 `Cherry Studio` 中设置的提供商类型 `OpenAI Response` 需要修改为 `OpenAI`。

![](https://image.heavenroad.org/48892cc3ce1e3c4379b694afa45c5d35.png)

添加模型 `NanoBanana` (即 `gemini-2.5-flash-image-preview`)。

![](https://image.heavenroad.org/4674845c6412ec6f5366d109070047fc.png)

现在，在 `Cherry Studio` 中测试一下吧！

![](https://image.heavenroad.org/fdd35aa92224cd76cbf888ce3ff2cce2.png)

完美地满足了我们的要求，尽情享受“香蕉”吧！

### 注意事项

- 现阶段请避免在 `CLIProxyAPI` 中添加多个 Gemini Web 账户。因为当存在多个账户时，程序会轮询调用，这可能会破坏会話的连续性，导致请求失败。
- 在 `Cherry Studio` 中，**切勿**在 `OpenAI Response` 提供商类型下添加 `NanoBanana` 模型。已知 `Cherry Studio` 在此情况下存在 Bug，会导致程序崩溃。

---

## 4. 中转转发接入

在前几篇文章中，我们已经成功通过 OAuth 或 Cookie 方式接入了 Qwen、Codex、Gemini CLI 和 Gemini Web。在本篇教程中，我们将更进一步，学习如何便捷地将各类 AI 中转服务接入 CLIProxyAPI。

首先，让我们回顾一下之前使用的配置文件：

```yaml
port: 8317

# 文件夹位置请根据你的实际情况填写
auth-dir: "Z:\\CLIProxyAPI\\auths"

request-retry: 3

quota-exceeded:
  switch-project: true
  switch-preview-model: true

api-keys:
# Key请自行设置，用于客户端访问代理
- "ABC-123456"
```

初次配置后，我们一直没有改动过它。现在，是时候对这个文件进行一些扩展了。

我们先来添加一个 Claude 的中转服务。为此，我们首先需要获取该服务的 `base-url`，这个地址通常可以在相应服务商的官方文档或教程中找到。

以 88code 为例，在其官方教程中可以找到如下信息：

![](https://image.heavenroad.org/11c41d79d62c02df1ac5d5998c75d3e5.png)

从图中可以获知，88code 中转 Claude 服务的 `base-url` 是 `https://www.88code.org/api`。

我们在配置文件中加入 `claude-api-key` 字段：

```yaml
port: 8317
auth-dir: "Z:\\CLIProxyAPI\\auths"
request-retry: 3
quota-exceeded:
  switch-project: true
  switch-preview-model: true
api-keys:
- "ABC-123456"

claude-api-key:
  - api-key: "88_XXXXXXXXXXXXXXXXXXXXXXXXX"
    base-url: "https://www.88code.org/api"
```

同样地，88code 也提供了 Codex 服务。我们依照相同的方法，找到其 `base-url`：

![](https://image.heavenroad.org/28e5ce297bca540e052863860dd9eb2c.png)

然后，在配置文件中添加 `codex-api-key` 字段：

```yaml
port: 8317
auth-dir: "Z:\\CLIProxyAPI\\auths"
request-retry: 3
quota-exceeded:
  switch-project: true
  switch-preview-model: true
api-keys:
- "ABC-123456"

claude-api-key:
  - api-key: "88_XXXXXXXXXXXXXXXXXXXXXXXXX"
    base-url: "https://www.88code.org/api"
    
codex-api-key:
  - api-key: "88_XXXXXXXXXXXXXXXXXXXXXXXXX"
    base-url: "https://www.88code.org/openai/v1"
```

对于其他服务商，也可以采用类似的方式进行添加。例如，我这里还有几个 PackyCode 的 Codex API Key，我将它们一并加入配置：

```yaml
port: 8317
auth-dir: "Z:\\CLIProxyAPI\\auths"
request-retry: 3
quota-exceeded:
  switch-project: true
  switch-preview-model: true
api-keys:
- "ABC-123456"

claude-api-key:
  - api-key: "88_XXXXXXXXXXXXXXXXXXXXXXXXX"
    base-url: "https://www.88code.org/api"
  - api-key: "sk-4cXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
    base-url: "https://api.packycode.com"
  - api-key: "sk-HpYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY"
    base-url: "https://api.packycode.com"

codex-api-key:
  - api-key: "88_XXXXXXXXXXXXXXXXXXXXXXXXX"
    base-url: "https://www.88code.org/openai/v1"
  - api-key: "fk-4cXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
    base-url: "https://oai-api.fkclaude.com/v1"
  - api-key: "sk-amXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
    base-url: "https://codex-api.packycode.com/v1"
  - api-key: "sk-sTXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
    base-url: "https://codex-api.packycode.com/v1"
```

请注意，即使是同一服务商、使用相同 `base-url` 的多个 `api-key`，也需要为每一条 `api-key` 单独声明 `base-url`，不可省略。

此外，CLIProxyAPI 还支持接入任何兼容 OpenAI 接口的供应商，这需要通过 `openai-compatibility` 字段来配置。在此不再赘述具体步骤，大家可以直接参考下方的配置文件示例进行配置：

```yaml
port: 8317
auth-dir: "Z:\\CLIProxyAPI\\auths"
request-retry: 3
quota-exceeded:
  switch-project: true
  switch-preview-model: true
api-keys:
- "ABC-123456"

claude-api-key:
  - api-key: "88_XXXXXXXXXXXXXXXXXXXXXXXXX"
    base-url: "https://www.88code.org/api"

codex-api-key:
  - api-key: "88_XXXXXXXXXXXXXXXXXXXXXXXXX"
    base-url: "https://www.88code.org/openai/v1"
  - api-key: "fk-4cXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
    base-url: "https://oai-api.fkclaude.com/v1"
  - api-key: "sk-amXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
    base-url: "https://codex-api.packycode.com/v1"
  - api-key: "sk-sTXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
    base-url: "https://codex-api.packycode.com/v1"

openai-compatibility:
  - name: "openrouter"
    base-url: "https://openrouter.ai/api/v1"
    api-keys:
      - "sk-or-v1-aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
      - "sk-or-v1-bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
    models:
      - name: "deepseek/deepseek-chat-v3.1:free"
        alias: "deepseek-v3.1"
      - name: "deepseek/deepseek-r1-0528:free"
        alias: "deepseek-r1-0528"
      - name: "x-ai/grok-4-fast:free"
        alias: "grok-4-fast"
  - name: "groq"
    base-url: "https://api.groq.com/openai/v1"
    api-keys:
      - "gsk_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
    models:
      - name: "deepseek-r1-distill-llama-70b"
        alias: "deepseek-r1-70b"
```

可以看到，`openai-compatibility` 的配置逻辑与之前略有不同：同一供应商（Provider）下的所有 `api-key` 共享同一个 `base-url`。

至此，配置就完成了。剩下的模型连通性验证，就留给各位读者自行测试了。

---

## 5. Docker 服务器部署

在之前的系列文章中，我们介绍了如何在本地电脑上使用 CLIProxyAPI。本文将更进一步，讲解如何在服务器上通过 Docker 完成部署。

### **一、 环境准备**

在开始之前，请确保你拥有一台可用的 VPS（虚拟专用服务器）。本文将以 **Debian 13** 系统为例进行演示。

同时，请确保你的服务器上已经安装了 **Git** 和 **Docker**。

如果尚未安装，可以通过以下命令进行安装：

**1. 安装 Git**
```bash
apt update && apt install git -y
```

**2. 安装 Docker**

可以使用官方提供的一键脚本进行安装：

```bash
bash <(curl -fsSL https://get.docker.com)
```

### **二、 部署 CLIProxyAPI**

准备工作就绪后，请依次执行以下命令来克隆项目并初始化配置。

```bash
git clone https://github.com/luispater/CLIProxyAPI.git
cd CLIProxyAPI
cp config.example.yaml config.yaml
```

![](https://image.heavenroad.org/60714b62a0f5b3ea896ab5461ecec150.png)

此时，我们可以打开 `config.yaml` 文件进行编辑。本教程将采用以下最小化配置作为示例：

```yaml
port: 8317

# 文件夹位置请根据你的实际情况填写
auth-dir: "~/.cli-proxy-api"

request-retry: 3

quota-exceeded:
  switch-project: true
  switch-preview-model: true

api-keys:
# Key 请自行设置，用于客户端访问代理
- "ABC-123456"
```

**请注意：** 当使用 Docker 部署时，建议保持 `auth-dir` 的默认设置，无需修改。

编辑完 `config.yaml` 文件后，我们执行以下命令来执行 Docker 容器构建脚本。

```bash
bash docker-build.sh
```

脚本会提供两个选项：

![](https://image.heavenroad.org/f0f543ef4004f3f81c029f442b54c8bc.png)

- **选项 1：** 直接使用 Docker Hub 上的预构建镜像运行 (`docker compose up -d`)，速度快。
- **选项 2：** 在服务器本地编译镜像然后再运行，适合需要自定义修改的场景。

在本教程中，我们选择**选项 1**，以快速启动服务，稍待片刻，服务便成功启动了。

![](https://image.heavenroad.org/44609a9a0746c138f570202bd2825366.png)

### **三、 查看日志**

尽管脚本提示使用 `docker compose logs -f` 查看日志，但由于程序默认会将日志重定向到文件，因此**实时查看日志**需要使用以下命令：

```bash
tail -f ./logs/main.log
```

### **四、 添加 OAuth 认证**

现在程序已经在正常运行了。如果需要添加中转 Key，只需按照之前文章介绍的方法编辑配置文件即可，这一次，我们重点讲解如何通过 OAuth 添加授权认证文件。

#### **步骤一：在服务器端生成认证链接**

以添加 **Codex** 为例，请在项目根目录下执行以下命令：

```bash
docker compose exec cli-proxy-api /CLIProxyAPI/CLIProxyAPI -no-browser --codex-login
```

程序会生成一段用于建立 SSH 隧道的命令，请复制箭头处 `ssh` 开头的整段命令。

![](https://image.heavenroad.org/42f152ef068cea1df603b29934b6e814.png)

#### **步骤二：在本地建立 SSH 隧道**

在**你自己电脑**的终端或命令行工具中，粘贴刚才复制的命令。

![](https://image.heavenroad.org/1c27a2d2e3bc1ad823a040a50cb8e143.png)

**特别注意：** 需要将命令中 `-p` 参数后的端口号（示例中的 `22`）替换为你 VPS 的**实际 SSH 端口**。

回车后输入服务器的 SSH 登录密码。成功连接后，请保持此终端窗口不要关闭，然后回到刚才操作服务器的终端上。

![](https://image.heavenroad.org/e25d23bc9b129cc9bbc6f4e1a674fed5.png)

#### **步骤三：通过浏览器完成授权**

复制服务器终端上箭头指向的链接

![](https://image.heavenroad.org/5ddcbc39551201f61b906703034af3d8.png)

在你本地电脑的浏览器中打开这个链接，用你的 ChatGPT 账号登录并授权

![](https://image.heavenroad.org/a4ed389a080bce529c47bda6f3129189.png)

授权成功后，会看到如下画面：

![](https://image.heavenroad.org/507c93b6fc900eae5c6c5b486b399561.png)

同时，服务器的终端上也会显示认证文件已成功保存

![](https://image.heavenroad.org/a34bb04a63f7f75f687a2a626035dccd.png)

至此，Codex 的认证就全部完成了。对于 Gemini-CLI 和 Claude 等其他需要 OAuth 授权的服务，操作流程完全相同。

### **五、 原理总结**

最后，我们来总结一下这个远程 OAuth 认证流程的原理：

Gemini-CLI、Claude 和 Codex 的 OAuth 认证都需要一个“回调”（Callback）过程来接收授权令牌。由于安全限制，服务商的回调地址通常强制设置为 `localhost`。

当我们在 Docker 容器中执行授权命令时，容器内没有浏览器环境，我们必须在本地电脑上打开授权网页。但授权成功后，浏览器会尝试访问 `localhost`，这只会访问到我们自己的电脑，而无法将令牌传递给远在服务器上的程序。

**SSH 隧道（SSH Tunnel）** 的作用就是搭建一座桥梁：它将我们本地电脑的某个端口（例如 `1455`）上的所有网络请求，通过加密的 SSH 连接，转发到服务器的同一端口上。这样，当浏览器访问本地的 `http://localhost:1455` 时，请求实际上被转发给了服务器上正在监听 `1455` 端口的 CLIProxyAPI 程序，从而巧妙地完成了远程认证。

作为 SSH Tunnel 的替代方案，你也可以在浏览器跳转到 `localhost` 回调链接时，手动将其中的 `localhost` 替换为你服务器的 IP 或域名。不过，请注意，这种方法需要你正确配置服务器的防火墙或反向代理，以确保回调请求能够被正确接收，否则可能会认证失败。

### **六、 客户端使用**

完成以上配置后，在客户端使用时，只需将请求的端点（Endpoint）地址指向你服务器的 `IP:端口`（例如 `http://YOUR_SERVER_IP:8317`）即可，其余操作与本地使用完全相同。

至此，你已经掌握了在服务器上通过 Docker 部署 CLIProxyAPI 的完整流程，快去享受 AI 带来的便利吧！

如果在配置或使用过程中遇到任何问题，欢迎加入官方社群进行交流咨询，作者很热情很 Nice 哟~

> [!note]
> 转载自@hkfires 发布于 9/29/2025, 11:09:36 AM，编辑于 9/29/2025, 11:13:04 AM
>
> - [手把手带你用上AI神器 - CLIProxyAPI（零：配置详细解说）](https://www.nodeseek.com/post-464540-1)
> - [手把手带你用上AI神器 - CLIProxyAPI（壹：项目介绍+Qwen实战）](https://www.nodeseek.com/post-464541-1)
> - [手把手带你用上AI神器 - CLIProxyAPI（贰：Gemini CLI+Codex实战）](https://www.nodeseek.com/post-464542-1)
> - [手把手带你用上AI神器 - CLIProxyAPI（叁：NanoBanana实战）](https://www.nodeseek.com/post-464544-1)
> - [手把手带你用上AI神器 - CLIProxyAPI（肆：中转转发接入篇）](https://www.nodeseek.com/post-464546-1)
> - [手把手带你用上AI神器 - CLIProxyAPI（伍：Docker服务器部署）](https://www.nodeseek.com/post-464547-1)
> - QQ 群：`188637136` | Telegram 群：[https://t.me/CLIProxyAPI](https://t.me/CLIProxyAPI)
