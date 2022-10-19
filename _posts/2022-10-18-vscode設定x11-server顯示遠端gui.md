---
layout: post
title: VSCode設定X11 Server顯示遠端GUI
date: 2022-10-18 11:27 +0800
categories: [環境設定與部屬]
tags: [Linux]
---


參考:  
https://dreamanddead.github.io/post/ssh-x11-forward/  

https://juejin.cn/post/7009593663894323231  

https://stackoverflow.com/questions/65468655/vs-code-remote-x11-cant-get-display-while-connecting-to-remote-server  

https://zhuanlan.zhihu.com/p/461378596  

X11 Forwarding on Windows with Cygwin  
https://www.csusb.edu/sites/default/files/cse_x11_forwarding_on_windows_with_cygwin.pdf  

Cygwin/X  
https://x.cygwin.com/  

ssh option  
https://www.microfocus.com/documentation/rsit-server-client-unix/8-4-0/unix-guide/ssh_options_ap.html 


connect localhost port 6000  
https://www.cs.odu.edu/~zeil/cs252/latest/Public/xtrouble/index.html  

What exactly is X/Xorg/X11?
https://www.reddit.com/r/linuxquestions/comments/3uh9n9/what_exactly_is_xxorgx11/

SSH X11 Forwarding Of Gnome-Boxes Under Wayland & Mixed Wayland & X11 Environments
https://www.dbts-analytics.com/notesxfwdgb.html



* X11 forwarding docker container裡面的影像
docker run 指令要多加  --volume="$HOME/.Xauthority:/root/.Xauthority:rw"


X11 forwarding of a GUI app running in docker
https://stackoverflow.com/a/51209546




Run X application in a Docker container reliably on a server connected via SSH without "--net host"
https://stackoverflow.com/questions/48235040/run-x-application-in-a-docker-container-reliably-on-a-server-connected-via-ssh-w