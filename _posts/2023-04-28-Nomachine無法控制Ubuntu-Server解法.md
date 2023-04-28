---
layout: post
title: Nomachine無法控制Ubuntu Server解法
date: 2023-04-28 17:28 +0800
categories: [環境設定與部屬]
tags: [ubuntu]     # TAG names should always be lowercase
---


1. 無螢幕的Server(Headless server)
有三種方法
* 設定x server接受headless狀態
* 插上一個假的HDMI dongle
* 讓Nomachine建立虛擬螢幕
對server下指令關閉display-manager並且重啟Nomachine服務
```bash
sudo systemctl stop display-manager
sudo /etc/NX/nxserver --restart
```

參考:
https://forum.nomachine.com/topic/problems-connecting-with-nomachine-in-pc-client-without-monitor