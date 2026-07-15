---
title: 水货三星 Z Flip 5刷机
category: Blog
tags:
  - 手机
  - 刷机
  - 三星
published: 2026-03-26T08:03:29+08:00
slug: slug_20260326080329
image: https://www.techadvisor.com/wp-content/uploads/2024/04/Galaxy-Z-Flip-5-review_1.jpg
Last Modified: 2026-07-15 11:07:12
---
黄鱼淘的二手 Z flip 5，拿到发现解锁了 boot loader，系统显示 SM-F7310 （OneUI 6.1.1 Android 14）是国行的系统，我想更新 one ui 7 但是刷 7310 的系统提示 PIT 错误。

去 setting - about - software 看它的详细信息
```
baseband version: F731BXXS3DXI9
BUILD NUMBER FLIP5_ROM_V5.0
SE for Android status: Enforing SEPF_SM-F7310_13_0001
Service provider software version: SAOMC_SM-F7310_ZAC_CHC_14_0008 CHC/CHC/THL
```
gemini 认为这个机器可能是泰国版 (THL) 被强行刷了魔改的中国版（FLIP5_ROM_V5.0）
于是去找 F731B 的 firmware - https://samfw.com/firmware/SM-F731B
找到以后挑个顺眼的国家（我选了 EUX），刷机成功。

后话
和港版一样，内置谷歌全家桶的固件刷机后的激活必须连 google，那么就要在局域网内部署软路由网关。我是用的旁路由，在激活连 wifi 的时候设置把 IP 地址的网关指向旁路由。激活成功。

因为商家为了刷国行，已经 unlock 了 bootloader, 所以破坏了硬件保险丝，再 lock 意义不大，风险却很大。所以没有再 lock bootloader，缺点是 samsung pay 和安全文件夹无法使用（反正也不用）。
