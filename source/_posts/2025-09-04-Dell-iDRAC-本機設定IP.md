---
title: Dell iDRAC 本機設定IP
date: 2025-09-04 10:45:54
categories:
tags:
---
先安裝racadm，在輸入service tag後的軟體下載頁面`適用於 Microsoft Windows Server(R) 的 Dell EMC iDRAC 工具，v11.0.0.0`
Dell-iDRACTools-Web-WINX64-11.3.0.0-609_A00


```shell
#查IP
racadm get iDRAC.IPv4
#設定IP
racadm set iDRAC.IPv4.Address 192.168.0.1
racadm set iDRAC.IPv4.Netmask 255.255.255.0
racadm set iDRAC.IPv4.Gateway 192.168.0.254
```


在作業系統打開iDrac網頁

1. 下載對應型號的iDRAC Service Module for Windows
2. 按iDRACSvcMod進行安裝
3. 在開始用iDRAC UI Launcher打開網頁 


https://www.dell.com/support/kbdoc/zh-tw/000194572/poweredge-%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%9D-ism-%E5%8F%8A%E5%95%9F%E5%8B%95-idrac-ui-%E5%95%9F%E5%8B%95%E5%99%A8