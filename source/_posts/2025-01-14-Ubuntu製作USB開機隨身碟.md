---
title: Ubuntu製作USB開機隨身碟
date: 2025-01-14 10:42:44
categories:
tags:
---


```
lsblk
sudo umount /dev/sd<?><?>  
sudo dd bs=4M if=path/to/input.iso of=/dev/sd<?> conv=fdatasync  status=progress
```

參考:
https://askubuntu.com/a/377561
