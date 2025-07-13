---
title: 【CS日志】linux快速科学上网
published: 2025-06-27
description: ''
image: ''
tags:
  - journal
category: CS心得
draft: false
lang: ''
---
鸣谢：感谢 [@Xecades](https://github.com/xecades)老师对本方法的原创性贡献！本文仅仅是转述他的方法。

---

### 环境：
- MacBook M3 Pro
- 远程服务器(我自己是ubuntu系统docker)

### 步骤：
1. 反向映射（在Mac本机）

假设你的macbook代理端口（HTTP）号是7890（可以打开ClashX设置确认）

远程服务器用户ip：`root@XX.XXX.XXX.XXX`。
```bash
# tilly @ tillydeMacBook-Pro in ~ [16:50:16]
$ ssh -p 11135 -NfR 7890:127.0.0.1:7890 root@XX.XXX.XXX.XXX

root@XX.XXX.XXX.XXX's password: 
(base)
```
然后ssh登陆服务器：
```bash
ssh -p 11135 -L 2020:localhost:2017 root@XX.XXX.XXX.XXX
```

2. 设置代理变量

原始做法是
```bash
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
```
可以写入`.bashrc`别名方便使用：
```bash
echo "alias enableproxy='export http_proxy=http://127.0.0.1:7890 && export https_proxy=http://127.0.0.1:7890'" >> ~/.bashrc
echo "alias disableproxy='unset http_proxy https_proxy all_proxy'" >> ~/.bashrc
source ~/.bashrc
```
以后只需：
```bash
enableproxy   # 打开代理
disableproxy  # 关闭代理
```


如果用的`tmux`新窗口，需要打开窗口使用，在外面配置是没用的。

3. 跑通
```bash
(base) root@User:~# enableproxy
(base) root@User:~# curl https://www.google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>302 Moved</TITLE></HEAD><BODY>
<H1>302 Moved</H1>
The document has moved
<A HREF="https://www.google.com.hk/url?XXXXXX">here</A>.
</BODY></HTML>
(base) root@User:~#
```
出现`302 Moved`说明转移成功，配置完成。

PS：
1. 若出现`bind [127.0.0.1]:2017: Address already in use`说明端口被占用，比如我原来是`2017`，后面就换了`2020`。
2. 每次登录服务器都记得完整做一遍上面的步骤，包括最开始的本机mac转发，不然在服务器上`enableproxy`会显示没有这个bash命令。
3. 有的服务器是用falcon登录：
```bash
ssh -NfR 7890:127.0.0.1:7890 falcon

ssh -L 2021:localhost:2017 falcon
# 然后记得改.bachrc配置文件
```

---

补充一点遇到的问题：
1. 端口被占用
可以指定命令端口，如`port 29501`：
```bash
CUDA_VISIBLE_DEVICES=0 torchrun --nproc_per_node 1 --rdzv_backend=c10d --rdzv_endpoint=localhost:29501 run.py args/gsm_coconut.yaml
```
2. 没有权限访问hf上的公开模型
需要用token进行设备登录。
先在 https://huggingface.co/settings/tokens 上注册一个read权限token（初创记得复制），然后
```bash
huggingface-cli login
```
输入token，若有
```bash
Login successful.
```
说明登录成功。