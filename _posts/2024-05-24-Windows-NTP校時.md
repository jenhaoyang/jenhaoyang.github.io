---
layout: post
title: Windows-NTP校時
date: 2024-05-24 17:31 +0800
---

執行 regedit
執行 

https://www.ravenswoodtechnology.com/network-time-protocol-configurations-a-deeper-dive/

https://www.ravenswoodtechnology.com/in-sync-proper-time-configuration-in-ad/


https://superuser.com/questions/451018/how-can-i-query-an-ntp-server-under-windows


https://www.renanrodrigues.com/post/how-to-configure-ntp-server-in-active-directory-step-by-step


Windows作為NTP server
https://support.hanwhavision.com/hc/en-us/articles/26570683589529-How-to-Setup-an-NTP-Server-on-Windows-10


w32tm /query /configuration
確認看到以下訊息，(本機)代表是從registry讀取的設定，(原則)代表是從 policy(GPO) 讀取的設定
```shell
NtpServer (本機)
DllName: C:\Windows\SYSTEM32\w32time.DLL (本機)
Enabled: 1 (原則)
InputProvider: 0 (本機)
AllowNonstandardModeCombinations: 1 (本機)
```

```