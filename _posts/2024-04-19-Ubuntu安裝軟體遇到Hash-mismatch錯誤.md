---
layout: post
title: Ubuntu安裝軟體遇到Hash-mismatch錯誤
date: 2024-04-19 17:31 +0800
---

* 如果是在wsl，檢查一下nameserver設定，確認DNS伺服器有沒有正確設定。
  https://geekdudes.wordpress.com/2022/11/16/windows-subsistem-for-linux-make-etc-resolv-conf-changes-permanent/

* 在網路很糟的情況下可以重複`apt update`，有mismatch的部分會慢慢被校正。

* 更改鏡像網站
https://note.drx.tw/2012/01/mirror.html  

  
https://blog.miniasp.com/post/2019/02/19/Ubuntu-apt-update-hash-sum-mismatch#google_vignette