---
title: 【CS日志】跑模型常见报错
published: 2025-07-14
description: ''
image: ''
tags:
  - journal
category: CS日志
draft: false
lang: ''
---

```bash
ValueError: Total number of attention heads (28) must be divisible by tensor parallel size (3).
```
报错原因：模型注意力头无法被GPU数量整除，比如28不能被3个GPU均分，需要设置GPU数量为2或4等。

---
