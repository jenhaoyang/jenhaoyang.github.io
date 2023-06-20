---
layout: post
title: Deepstream辨識結果遠端觀看
date: 2023-04-28 17:28 +0800
categories: [伺服器管理]
tags: [ubuntu]     # TAG names should always be lowercase
---

# RTSP遠端觀看設定
可以參考[這篇](https://forums.developer.nvidia.com/t/jetson-nano-faq/82953)
特別注意如果Deepstream不是在Jetson上面跑記得要換成下面指令啟動server，因為dGPU的Deepstream沒有`nvvidconv`原件!!

```shell
./test-launch "videotestsrc is-live=1 ! videoconvert  ! x264enc ! h264parse ! rtph264pay name=pay0 pt=96"
```
# 整合到自己的程式裡
`sources\apps\apps-common\src\deepstream_sink_bin.c`有詳細的整合方式。

https://github.com/aler9/mediamtx#installation






