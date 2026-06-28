---
title: Clash Meta组合自建和机场的节点配置
category: VPS
tags:
  - clash
  - 节点
  - 机场
published: 2026-06-28T18:06:85+08:00
image: https://image.heavenroad.org/default_cover.webp
slug: slug20260628180685
upload: false
Last Modified: 2026-06-28 18:06:12
---

## 前言
把自建节点和机场组合在一起，使用自己的分流规则甚至是链式落地

1. 找一个 clash meta 配置模板
2. 把自建节点配置好
3. 添加机场分 3 步
	1. 获取机场订阅 url
	2. 新建一个 proxy-provider
	3. 在 proxy-group 中使用

## 获取机场订阅 url

从各机场中复制订阅链接，比如 [良心云aff](https://xn--9kqz23b19z.com/#/register?code=GJSzpPbf)

## 新建一个 proxy-provider

```
proxy-providers:

Airport_LXY:
	type: http
	path: ./providers/Airport_LXY.yaml
	url: "https://这里替换成自己的订阅链接"
	interval: 3600
	
	health-check:
		enable: true
		interval: 180
		url: http://www.google.com/generate_204
		timeout: 1000
		lazy: false
		expected-status: 204
```

如果要进一步过滤特定节点比如香港，可以在 `interval` 下加一行 `filter: "(?i)港|hk|hongkong|hong kong"`

## 在 proxy-groups 中使用

在 `proxy-groups` 里面新建一个

```
- name: 🩵良心云自动
	type: url-test
	url: http://www.google.com/generate_204
	interval: 60
	use:
		- Airport_LXY
	lazy: false
```

如果你有一个香港的 proxy-group，并且前面写好了带 filter 的 provider 叫 `Airport_LXY_HK`，可以在自建的香港节点后加一个 `use` ，clash 会自动合并到一起，也支持合并多个机场订阅

```
- name: 🇭🇰香港
	type: url-test
	proxies:
	- 自建1
	- 自建2
	url: http://cp.cloudflare.com/generate_204
	use:
		- Airport_LXY_HK
		- Airport_XXX_HK
	interval: 300
	lazy: true
```

## 用机场链式代理自己的落地

假设你的机场支持的情况下，你又想用自己的落地节点，新建一个节点并在最后加一个 `dialer-proxy: 🩵良心云自动` 即可。（推荐把 `🩵良心云自动` 的类型采用 url-test，否则可能经常需要手动切换节点）

```
- name: 自家落地@🩵良心云自动
type: ss
server: x.x.x.x
port: xxx
cipher: 2022-blake3-aes-256-gcm
password: "xxxxxxx"
udp: true
dialer-proxy: 🩵良心云自动
```
