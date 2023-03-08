---
layout: post
title: deepstream教學
date: 2023-01-11 12:28 +0800
categories: [深度學習工具]
tags: [deepstream]     # TAG names should always be lowercase
---

# 畫出範例的結構圖
首先在`~/.bashrc`加入下面這行設定pipeline圖儲存的位置，注意GStreamer不會幫你建立資料夾，你必須確認資料夾存在
```shell
export GST_DEBUG_DUMP_DOT_DIR=/tmp
```
接下來在pipeline 狀態設為PLAYING之前加入下面這行程式
```c
GST_DEBUG_BIN_TO_DOT_FILE(pipeline, GST_DEBUG_GRAPH_SHOW_ALL, "dstest1-pipeline");
```
最後執行程式後就會產生`.dot`在前面設定的資料夾，你可以下載[Graphviz](http://www.graphviz.org/)，或是用[VScode的插件](https://marketplace.visualstudio.com/items?itemName=tintinweb.graphviz-interactive-preview)來看圖


## Deepstream 說明書
https://docs.nvidia.com/metropolis/deepstream/dev-guide/text/DS_plugin_gst-nvdsxfer.html

## gst-launch-1.0建立rtsp輸入源的pipeline
首先先用`gst-launch-1.0`建立一個簡單的rtsp輸入、螢幕輸出的pipeline

```
gst-launch-1.0 rtspsrc location='rtsp://192.168.1.10:554/user=admin_password=xxxxxx_channel=1_stream=0.sdp' ! rtph264depay ! h264parse ! nvv4l2decoder ! nvvideoconvert ! video/x-raw,format=BGRx ! videoconvert ! video/x-raw,format=BGR ! autovideosink
```

## 將python的範例程式轉成c++
https://github.com/NVIDIA-AI-IOT/deepstream_python_apps/tree/master/apps/deepstream-rtsp-in-rtsp-out


## 建立mjpeg串流
1. 指令方式
```
gst-launch-1.0 -v rtspsrc location="rtsp://<rtsp url>/live1.sdp" \
! rtph264depay ! avdec_h264 \
! timeoverlay halignment=right valignment=bottom \
! videorate ! video/x-raw,framerate=37000/1001 ! jpegenc ! multifilesink location="snapshot.jpeg"
```
https://stackoverflow.com/questions/59885450/jpeg-live-stream-in-html-slow


## 查詢deepstream bin的說明
```
gst-inspect-1.0 nvurisrcbin
```

## gst-launch-1.0輸出除錯訊息到檔案
參考:  
https://www.cnblogs.com/xleng/p/12228720.html
```
GST_DEBUG_NO_COLOR=1 GST_DEBUG_FILE=pipeline.log GST_DEBUG=5 gst-launch-1.0 -v rtspsrc location="rtsp://192.168.8.19/live.sdp" user-id="root" user-pw="3edc\$RFV" ! rtph264depay ! avdec_h264 ! timeoverlay halignment=right valignment=bottom ! videorate ! video/x-raw,framerate=37000/1001 ! jpegenc ! multifilesink location="snapshot.jpeg"
```

參考:
https://gstreamer.freedesktop.org/documentation/tutorials/basic/debugging-tools.html?gi-language=c  
https://embeddedartistry.com/blog/2018/02/22/generating-gstreamer-pipeline-graphs/  

## 本地端觀看udp傳送影像
host設為本機ip或127.0.0.1

send:
```
gst-launch-1.0 -v videotestsrc ! x264enc tune=zerolatency bitrate=500 speed-preset=superfast ! rtph264pay ! udpsink port=5000 host=$HOST
```

receive:
```
gst-launch-1.0 -v udpsrc port=5000 ! "application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H264, payload=(int)96" ! rtph264depay ! h264parse ! avdec_h264 ! videoconvert ! autovideosink
```

## Glibs說明書
http://irtfweb.ifa.hawaii.edu/SoftwareDocs/gtk20/glib/glib-hash-tables.html#g-int-hash

## GDB文字圖形介面
https://blog.louie.lu/2016/09/12/gdb-%E9%8C%A6%E5%9B%8A%E5%A6%99%E8%A8%88/

## 範例
https://gist.github.com/liviaerxin/bb34725037fd04afa76ef9252c2ee875#tips-for-debug

## rtsp 元件nvrtspoutsinkbin
nvrtspoutsinkbin沒有說明書，只能用gst-inspect-1.0看

## 設定source id
https://forums.developer.nvidia.com/t/how-to-get-sources-index-in-deepstream/244461  

## 切換輸入源
https://forums.developer.nvidia.com/t/how-switch-camera-output-gst-nvmultistreamtiler/233062

```
tiler_sink_pad.add_probe(Gst.PadProbeType.BUFFER, tiler_sink_pad_buffer_probe, 0)

tiler.set_property("show-source", <stream_id>) `
```

deepstream_source_yaml.cpp有範例

參考:  
https://www.gclue.jp/2022/06/gstreamer.html  