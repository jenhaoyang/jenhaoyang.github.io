---
layout: post
title: Nomachine無法控制Ubuntu Server解法
date: 2023-04-28 17:28 +0800
categories: [環境設定與部屬]
tags: [ubuntu]     # TAG names should always be lowercase
---


1. 無螢幕的Server(Headless server)並且有顯卡，登入後螢幕是黑的
https://kb.nomachine.com/AR03P00973 根據步驟1的做法

```bash
sudo systemctl stop display-manager
sudo /etc/NX/nxserver --restart
```




參考:
https://forum.nomachine.com/topic/problems-connecting-with-nomachine-in-pc-client-without-monitor