---
title: 【CS日志】服务器网络问题
published: 2025-06-27
description: ''
image: ''
tags:
  - journal
category: CS日志
draft: false
lang: ''
---

有时候服务器出现报错：
```bash
root@XXX:~# pip install --no-cache-dir fastNLP==0.7.0
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLEOFError(8, '[SSL: UNEXPECTED_EOF_WHILE_READING] EOF occurred in violation of protocol (_ssl.c:1010)'))': /simple/fastnlp/
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