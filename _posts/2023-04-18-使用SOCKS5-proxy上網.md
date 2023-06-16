---
layout: post
title: Windows使用SOCKS5-proxy上網
date: 2023-04-19 10:54 +0800
pin: true
---

# 簡介
這裡將介紹一個只需要使用SSH連線到一台可以連到對網網路的主機，就可以讓防火牆內的主機上網的方法
[圖片說明](https://ma.ttias.be/socks-proxy-linux-ssh-bypass-content-filters/)

# 設定putty
https://www.simplified.guide/putty/create-socks-proxy

# 或是用ssh指令連接到可以上網的電腦
```shell
ssh -D 4444 -q -C -N user@ma.ttias.be
```
# windows 設定SOCKS5 proxy

https://blog.gtwang.org/linux/ssh-tunnel-socks-proxy-forwarding-tutorial/

# 新增憑證
https://learn.microsoft.com/zh-tw/biztalk/adapters-and-accelerators/accelerator-swift/adding-certificates-to-the-certificates-store-on-the-client

# windows下設定git proxy command
https://serverfault.com/questions/956613/windows-10-ssh-proxycommand-posix-spawn-no-such-file-or-directory