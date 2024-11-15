---
layout: post
title: ssh-copy-id指令
date: 2022-06-24 23:12 +0800
categories: [環境設定與部屬]
tags: [ssh]
---

1. ssh-copy-id
* linux

* windows
在powershell中沒有ssh-copy-id這個指令，不過我們可以用下面指令達成相同目標
```shell
type $env:USERPROFILE\.ssh\id_rsa.pub | ssh {IP-ADDRESS-OR-FQDN} "cat >> .ssh/authorized_keys"
```

---
參考:
https://www.chrisjhart.com/Windows-10-ssh-copy-id/