---
title: 如何免费运行 Claude Code
category: Blog
tags:
  - ClaudeCode
  - Ollama
  - LLM
published: 2026-05-25T15:05:71+08:00
image: https://image.heavenroad.org/Pasted%20image%2020260525155159.png
slug: slug20260525150571
upload: false
Last Modified: 2026-05-25 15:05:46
---

在 AI 开发工具日益普及的今天，Claude Code 作为一个强大的代码助手（类似于一辆性能卓越的「汽车」），默认搭载的是 Anthropic 官方的强大「引擎」（如 Claude 3.7 Sonnet 或 Opus）。但是，持续调用这些闭源 API 的成本非常高昂。

幸运的是，我们完全可以「打开引擎盖」，将原本昂贵的闭源引擎替换为免费或极低成本的开源模型。今天，我们将详细介绍两种替代方案：使用本地运行的 Ollama 以及借助 OpenRouter 接入云端开源模型。

## 核心概念：开源模型 vs 闭源模型

在开始实操前，理解闭源与开源模型的区别非常重要。闭源模型（如 GPT-4o、Claude 3.7 Sonnet）通常性能更强、工具调用更稳定，但按 Token 收费且价格不菲。而开源模型（如 Qwen、Gemma）可以通过下载在本地免费运行，或者在云端以极低成本调用。

虽然开源模型在处理极为复杂的代码逻辑时可能略逊一筹，但它们与顶级闭源模型之间的性能差距正在快速缩小。对于日常的代码阅读、总结、文件检索、简单的脚手架（Scaffolding）搭建等任务，开源模型完全可以胜任。

## 方法一：使用 Ollama 在本地免费运行

这是最彻底的免费方案，模型完全运行在本地硬件上，隐私性极佳。

1. **安装 Ollama**：前往 Ollama 官网下载对应操作系统的安装包并安装。
    
2. **选择并下载模型**：建议参考 OpenRouter 上的编程模型跑分榜单。例如，可以选择性能不错的 Qwen 3.5（9B 参数版本）。在 VS Code 终端中运行命令进行下载，例如 `ollama pull qwen:9b`。
    
3. **配置并启动 Claude Code**：在终端中输入专属启动命令（如 `ollama launch claude`），此时会弹出一个选择列表，让你指定刚才下载好的本地模型。
    
4. **注意事项**：
    
    - 即使使用本地模型，首次登录 Claude Code 仍需绑定 Anthropic 账号并充值至少 5 美元以激活 API 权限。但由于后续调用的是本地模型，这 5 美元并不会被扣除。
        
    - 本地模型的运行速度取决于电脑配置（如 RAM 和 GPU）。如果遇到上下文丢失的问题，可以通过 Ollama 命令手动扩大模型的上下文窗口限制（Context Window）。

## 方法二：通过 OpenRouter 接入免费 / 廉价云端模型

如果本地电脑硬件配置一般，OpenRouter 提供了极佳的云端替代方案。它不仅汇集了海量模型，还提供了一批完全免费的开源模型接口。

1. **注册并充值 OpenRouter**：前往 OpenRouter 官网注册。为了解除每天 50 次免费调用的限制，建议充值 5 到 10 美元（这笔钱放在账户里不花，仅用于提升调用额度至每天 1000 次）。获取 OpenRouter 的 API 密钥。
    
2. **修改项目配置文件**：在代码项目中找到 `.claude/settings.local.json` 文件。
    
3. **替换 API 终端及模型配置**：
    
    - 将 API URL 指向 OpenRouter。
        
    - 将 Anthropic 的 Token 字段替换为 OpenRouter API 密钥。
        
    - **关键避坑点**：务必将配置文件中所有的模型选项（包括默认模型、Sonnet 模型、Haiku 模型等）全部手动修改为指定的 OpenRouter 免费模型。如果不把 Haiku 替换掉，Claude Code 在后台执行一些隐蔽的工具调用（如搜索文件）时，仍会默认调用官方的 Haiku 模型，导致意外扣费。
        
4. **启动测试**：配置完成后，再次打开 Claude Code 终端。此时，就可以享受到云端极速的响应，且不用花一分钱。

## 开源模型的局限性与适用场景

将 Claude Code 与开源模型结合是一把双刃剑。

- **局限性**：由于开源模型并未针对 Claude Code 的底层 JSON 工具调用进行过专门训练，它们在使用「网络搜索」等复杂工具时可能会报错或失败。
    
- **最佳适用场景**：适合低风险、高频次的任务。比如整理本地文件、总结超长代码库、批量阅读日志等。
    
- **终极性价比策略**：除了完全免费的模型外，也可以在 OpenRouter 上选择极低成本的优秀开源模型（如 Gemma 4 的 31B 版本）。这些模型极其聪明，但调用成本可能比 Claude Opus 便宜 50 到 100 倍！

无论选择本地化还是低成本云端方案，合理管理 Token 和上下文，都是高效利用 AI 辅助编程的关键。

---

[Ollama + Claude Code = 99% CHEAPER](https://www.youtube.com/watch?v=O2k_qwZA8HU)
