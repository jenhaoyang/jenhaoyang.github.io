---
layout: post
title: Ubuntu設定VNC遠端桌面
date: 2023-02-02 22:25 +0800
---

VNC使用ubuntu桌面
https://bytexd.com/how-to-install-configure-vnc-server-on-ubuntu/#google_vignette

VNC操作指令
http://kito.wikidot.com/vnc


清乾淨apt package
https://askubuntu.com/questions/187888/what-is-the-correct-way-to-completely-remove-an-application




# 刪除並重啟vnc server
```
vncserver -kill :1
vncserver -localhost no :1
```

參考:  
https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-vnc-on-ubuntu-20-04 