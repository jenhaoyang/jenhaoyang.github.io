---
layout: post
title: windows docker設定
date: 2023-04-19 10:54 +0800
---

# docker image搬離開系統槽
https://stackoverflow.com/questions/62441307/how-can-i-change-the-location-of-docker-images-when-using-docker-desktop-on-wsl2

1. 關閉wsl2
```bash
wsl --shutdown
```
2. 匯出docke檔案
```bash
wsl --export docker-desktop-data "D:\Docker\wsl\data\docker-desktop-data.tar"
```

3. unregister
```bash
wsl --unregister docker-desktop-data
```

4. 匯入
```bash
wsl --import docker-desktop-data "D:\Docker\wsl\data" "D:\Docker\wsl\data\docker-desktop-data.tar" --version 2
```


# wsl2 volume
https://stackoverflow.com/questions/61083772/where-are-docker-volumes-located-when-running-wsl-using-docker-desktop