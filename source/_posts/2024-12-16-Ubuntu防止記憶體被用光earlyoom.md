---
title: Ubuntu防止記憶體被用光earlyoom
date: 2024-12-16 11:02:49
categories:
tags: Ubuntu
---

查看earlyoom停止的程式
```shell
sudo journalctl -u earlyoom | grep sending
```

參考:  
https://github.com/rfjakob/earlyoom  
https://zhangjk98.xyz/early-oom-and-oomd-for-out-of-memory/  
