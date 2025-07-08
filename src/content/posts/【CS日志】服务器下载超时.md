---
title: 【CS日志】服务器下载超时
published: 2025-07-08
description: ''
image: ''
tags:
  - journal
category: CS心得
draft: false
lang: ''
---

有时下载过慢，pip install 依赖的时候会报错超时。
这个时候排除断网的问题，那就是网速太慢了。这种情况，比起增加时延，例如：
```bash
pip install --timeout=100 -r requirements.txt
```
我更喜欢
```bash
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple -r requirements.txt
```
如果想把清华源设置成持久配置，可以
```bash
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```