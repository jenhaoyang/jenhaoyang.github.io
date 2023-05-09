---
layout: post
title: 設定watchdog
date: 2023-05-09 17:28 +0800
categories: [Jetson]
tags: [環境設定與部屬]     # TAG names should always be lowercase
---

# 測試watchdog
L4T已經預設有watchdog，可以透過下面指令測試，他預設的機制是如果有人讀取`/dev/watchdog`這個檔案，他就會開始倒數計時，如果有人寫入檔案，則timer重置，下面指令將會讓watchgod重啟系統。
```shell
sudo tail -f /dev/watchdog
```
如果要避免重啟就必須寫入`/dev/watchdog`檔。或是結束`tail -f`

https://forums.developer.nvidia.com/t/configuring-watchdog-timer-on-tx1/44361/2?u=jenhao

參考:
官方文件下載區
https://developer.nvidia.com/embedded/downloads


