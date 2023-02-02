---
layout: post
title: Ubuntu設定VNC遠端桌面
date: 2023-02-02 22:25 +0800
---

1. 開防火牆
```shell
sudo ufw allow 5901/tcp
```
2. 安裝套件
```shell
sudo apt install xfce4 xfce4-goodies tightvncserver
```
3. 用sudo開啟VNC服務並設定密碼
```
sudo vncserver
```
4. 關閉session
```
vncserver -kill :1
```

5. 設定config
設定root的設定檔
```
nano /root/.vnc/xstartup
```

再最下面加入這行
```
startxfce4
```
然後重啟server
```
vncserver
```

6. autorun
設定自動啟動
```
sudo nano /etc/systemd/system/vncserver.service
```
加入下面指令
```
[Unit]
Description=TightVNC server
After=syslog.target network.target

[Service]
Type=forking
User=root
PAMName=login
PIDFile=/root/.vnc/%H:1.pid
ExecStartPre=-/usr/bin/vncserver -kill :1 > /dev/null 2>&1
ExecStart=/usr/bin/vncserver
ExecStop=/usr/bin/vncserver -kill :1

[Install]
WantedBy=multi-user.target
```

重載systemd
```
systemctl daemon-reload
```

enable自動載入
```
systemctl enable --now vncserver
```

7. 在Windows用TightVNC測試連線
下載[TightVNC](https://www.tightvnc.com/download.php)
並且輸入連線ip 
<ip>:5901


參考:  
https://serverspace.io/support/help/install-tightvnc-server-on-ubuntu-20-04/  