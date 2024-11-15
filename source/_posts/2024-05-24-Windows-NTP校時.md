---
layout: post
title: Windows-NTP校時
date: 2024-05-24 17:31 +0800
---

w32tm有兩個地方可以設定，一個是registry，一個是policy(GPO)
registry可以用regedit開啟，位於
`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W32Time`

policy用gpedit.msc可以開啟，位於
`Computer Configuration\Policies\Administrative Templates\System\Windows Time Service`

下面指令測試是否可以連線到NTP server
```shell
w32tm /stripchart /computer:time.windows.com /samples:5 
```

下面指令可以設定NTP server
```shell
 w32tm /config /manualpeerlist:NTP.iead.local /syncfromflags:manual /update
```

w32tm /query /configuration
確認看到以下訊息，(本機)代表是從registry讀取的設定，(原則)代表是從 policy(GPO) 讀取的設定
```shell
NtpServer (本機)
DllName: C:\Windows\SYSTEM32\w32time.DLL (本機)
Enabled: 1 (原則)
InputProvider: 0 (本機)
AllowNonstandardModeCombinations: 1 (本機)
```

執行 regedit
執行 

IBM重點參考文章
https://www.dell.com/support/kbdoc/zh-cn/000134430/windows-time-service-%E6%95%85%E9%9A%9C-%E5%A4%84%E7%90%86  



https://www.ravenswoodtechnology.com/network-time-protocol-configurations-a-deeper-dive/

https://www.ravenswoodtechnology.com/in-sync-proper-time-configuration-in-ad/


https://superuser.com/questions/451018/how-can-i-query-an-ntp-server-under-windows


https://www.renanrodrigues.com/post/how-to-configure-ntp-server-in-active-directory-step-by-step


Windows作為NTP server
https://support.hanwhavision.com/hc/en-us/articles/26570683589529-How-to-Setup-an-NTP-Server-on-Windows-10

