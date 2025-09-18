---
title: 【CS心得】搭博客
published: 2024-10-28T00:00:00.000Z
description: ''
image: ''
tags:
  - journal
category: CS日志
draft: false
lang: ''
---
***目录***

<!-- toc -->

- [重装依赖:](#%E9%87%8D%E8%A3%85%E4%BE%9D%E8%B5%96)

<!-- tocstop -->

### 重装依赖: 
删除 node_modules 文件夹和 package-lock.json（或 pnpm-lock.yaml）文件，然后重新安装依赖：
```zsh 
rm -rf node_modules
rm package-lock.json
pnpm install
```
（网络有连接问题就先用淘宝镜像源配置，再进行安装：
```zsh
pnpm set registry https://registry.npmmirror.com/）
```
(原源：)
```zsh
pnpm set registry https://registry.npmjs.org/
```
可以试试更新依赖项：
```zsh
pnpm outdated
```
有时候要清理pnpm缓存：
```zsh
pnpm store prune
```
最后commit work flow:
```zsh
git add .  
git commit -m "Update package.json and pnpm-lock.yaml" 
git push origin main
```
对文件目录下的所有.md文件自动运行命令markdown-toc -i 文件名.md（更新所有post目录）
```zsh
for file in *.md; do markdown-toc -i "$file"; done
```
