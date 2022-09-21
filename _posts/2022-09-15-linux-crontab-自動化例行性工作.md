---
layout: post
title: Linux crontab 自動化例行性工作
date: 2022-09-15 21:20 +0800
categories: [環境設定與部屬]
tags: [Linux]
---

# 指令
```
sudo crontab -e
01 14 * * * /home/joe/myscript >> /home/log/myscript.log 2>&1
```

參考:  
https://askubuntu.com/a/121560  
