---
layout: post
title: Proxy setting for UWP apps
categories: [Geek]
tags: [windows]
---

## UWP中代理服务器配置无效的解决方法

UWP App本身的沙箱安全机制默认是不允许loopback到本机的，需要解除限制。
打开命令行，进入UWP应用目录。
```bash
cd C:\Users\%username%\AppData\Local\Packages
```
找到自己的app，如OneNote目录名称是 Microsoft.Office.OneNote_8wekyb3d8bbwe
然后完成了.

---------------------------------------------

Ref: [https://www.zhihu.com/question/334349374?sort=created](https://www.zhihu.com/question/334349374?sort=created)
