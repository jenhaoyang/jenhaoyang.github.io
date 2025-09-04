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
racadm set iDRAC.IPv4.Address 10.179.104.203
racadm set iDRAC.IPv4.Netmask 255.255.255.0
racadm set iDRAC.IPv4.Gateway 10.179.104.254
```