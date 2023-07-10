---
layout: post
title: Ubuntu-cli-網路狀態檢查
date: 2023-05-3 17:28 +0800
categories: [網路除錯]
tags: [network]     # TAG names should always be lowercase
---


# 下載tracetcp
https://github.com/0xcafed00d/tracetcp

# 檢查遠端伺服器有沒有開port
```shell
nc -z -v -w5 <host> <port>
```
https://stackoverflow.com/a/9463554
