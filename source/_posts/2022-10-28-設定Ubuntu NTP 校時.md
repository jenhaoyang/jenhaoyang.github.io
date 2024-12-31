---
layout: post
title: Ubuntu NTP 校時
date: 2022-10-28 12:20 +0800
categories: [環境設定與部屬]
tags: [Ubuntu]
pin: true
---

# timedatectl 時間管理工具
* 顯示目前的設定狀態
```bash
# 顯示目前狀態
timedatectl
```

```bash
               Local time: 三 2023-04-26 11:00:22 CST
           Universal time: 三 2023-04-26 03:00:22 UTC
                 RTC time: 三 2023-04-26 03:00:22    
                Time zone: Asia/Taipei (CST, +0800)  
System clock synchronized: yes                       
              NTP service: active                    
          RTC in local TZ: no  
```

* 啟動網路校時
輸入下面指令後，就會打開網路校時功能
```bash
# 啟用 NTP 校時
timedatectl set-ntp yes
```
Ubuntu使用的是`systemd-timesyncd`，因此等一下要設定`systemd-timesyncd`

* 檢查`systemd-timesyncd`服務狀態
```bash
# 檢查 systemd-timesyncd 服務狀態
systemctl status systemd-timesyncd
```

```bash
● systemd-timesyncd.service - Network Time Synchronization
     Loaded: loaded (/lib/systemd/system/systemd-timesyncd.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2023-04-26 10:58:34 CST; 1min 19s ago
       Docs: man:systemd-timesyncd.service(8)
   Main PID: 221565 (systemd-timesyn)
     Status: "Synchronized to time server 91.189.89.198:123 (ntp.ubuntu.com)."
      Tasks: 2 (limit: 38317)
     Memory: 1.2M
     CGroup: /system.slice/systemd-timesyncd.service
             └─221565 /lib/systemd/systemd-timesyncd
```
* 設定校時伺服器
要設定校時伺服器可以用root權限編輯以下檔案`/etc/systemd/timesyncd.conf`

```bash
[Time]
# NTP 伺服器（以空白分隔多個伺服器）
NTP=tw.pool.ntp.org jp.pool.ntp.org

# 備用 NTP 伺服器（以空白分隔多個伺服器）
FallbackNTP=sg.pool.ntp.org ntp.ubuntu.com
```

* 重新啟動`systemd-timesyncd`服務
重啟服務讓更改生效
```bash
# 重新啟動 systemd-timesyncd 服務
systemctl restart systemd-timesyncd
```

* 檢查一下是不是有跟NTP校時了
用指令`systemctl status systemd-timesyncd`看一下目前服務狀態，如果有更新成功會顯示出來


# 錯誤排除
* 出現Server has too large root distance. Disconnecting.訊息
表示機器跟ntp server之間回應的時間太久，因此可以去修改`/etc/systemd/timesyncd.conf`並加入`RootDistanceMaxSec=`，通常30秒已經很夠用了
```shell
# See timesyncd.conf(5) for details.
[Time]
NTP=10.10.1.30
#FallbackNTP=
RootDistanceMaxSec=30
#PollIntervalMinSec=32
#PollIntervalMaxSec=2048
```

# timedatectl的NTP service為active的時候查看目前使用的是哪一個NTP服務
* systemd-timesyncd
```
systemctl status systemd-timesyncd.service
```
# ntpd 或 chronyd
```
ps -e | grep ntp
```

```
ps -e | grep chrony
```

# chronyc說明書
https://chrony-project.org/doc/4.6.1/chronyc.html


詳細說明如下
https://unix.stackexchange.com/a/655489

參考:  
https://www.cnblogs.com/pipci/p/12833228.html  
https://officeguide.cc/ubuntu-linux-timedatectl-time-synchronization-tutorial/  
https://www.digitalocean.com/community/tutorials/how-to-set-up-time-synchronization-on-ubuntu-20-04  
https://www.tenable.com/audits/items/CIS_Ubuntu_18.04_LTS_Server_v2.1.0_L1.audit:26286d27c59292cfdb9c7b04593edbed  
https://serverfault.com/questions/1024770/ubuntu-20-04-time-sync-problems-and-possibly-incorrect-status-information  
