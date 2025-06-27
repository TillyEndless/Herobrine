---
title: 【CS日志】服务器网络问题
published: 2025-6-27
description: ''
image: ''
tags:
  - journal
category: CS心得
draft: false
lang: ''
---

有时候服务器出现报错：
```bash

```
用下面的命令或许可以解决：
```bash
unset ALL_PROXY
unset all_proxy
unset http_proxy
unset https_proxy
unset HTTP_PROXY
unset HTTPS_PROXY
```
顺便检查一下🪜节点有没有掉了。