---
title: 【CS日志】重设github仓库历史
published: 2025-07-20
description: ''
image: ''
tags:
  - journal
category: CS心得
draft: false
lang: ''
---
```bash
git reflog
```
查看push/pull历史

常见消息格式（按q退出浏览）：
```bash
a4ebefb (HEAD -> main, origin/main) HEAD@{0}: reset: moving to a4ebefb
cbf9e37 HEAD@{1}: commit: cot配置文件
964671a HEAD@{2}: reset: moving to 964671a
a4ebefb (HEAD -> main, origin/main) HEAD@{3}: commit: cot非参数冻结逻辑扩展
276fee7 HEAD@{4}: commit: cot非参数冻结逻辑扩展
···
```
例如想要回退到某个时期刚提交的历史状态`cbf9e37 HEAD@{1}: commit: cot配置文件`：
```bash
git reset --hard cbf9e37
```
然后本地强制push：
```bash
git add .
git commit -m "git reflog reset"
git push --force-with-lease origin main
```

远程pull仓库强制pull：
```bash
# 第一步：獲取遠端的最新歷史
git fetch origin main

# 第二步：將本地分支強制重置成遠端的樣子
git reset --hard origin/main

git pull origin main
```

