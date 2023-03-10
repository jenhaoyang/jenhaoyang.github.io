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
https://forums.developer.nvidia.com/t/where-can-fine-nvrtspoutsinkbin-info/199124  

範例
/opt/nvidia/deepstream/deepstream/sources/apps/sample_apps/deepstream_reference_apps/deepstream-bodypose-3d/sources/deepstream_pose_estimation_app.cpp

```c
    /* Create RTSP output bin */
    rtsp_out_bin = gst_element_factory_make ("nvrtspoutsinkbin", "nvrtsp-renderer");

    if (!rtsp_out_bin) {
      g_printerr ("Failed to create RTSP output elements. Exiting.\n");
      return -1;
    }

    g_object_set (G_OBJECT (rtsp_out_bin), "sync", TRUE, NULL);
    g_object_set (G_OBJECT (rtsp_out_bin), "bitrate", 768000, NULL);
    g_object_set (G_OBJECT (rtsp_out_bin), "rtsp-port", rtsp_port_num, NULL);
    g_object_set (G_OBJECT (rtsp_out_bin), "enc-type", enc_type, NULL);

    gst_bin_add_many (GST_BIN (pipeline), rtsp_out_bin, NULL);
```

## 取得source id
[https://forums.developer.nvidia.com/t/how-to-get-sources-index-in-deepstream/244461  ](https://forums.developer.nvidia.com/t/can-i-pass-source-id-from-nvinferserver-along-with-frame/205418)

可以用prob取得meta data
deepstream_test3_app.c 有範例

[probe使用範例 ](https://forums.developer.nvidia.com/t/how-to-make-metadata-probe-for-classification-only-pipeline/171126) 

![metadata](/assets/img/deepstream/metadata.png)



## 切換輸入源
https://forums.developer.nvidia.com/t/how-switch-camera-output-gst-nvmultistreamtiler/233062

```
tiler_sink_pad.add_probe(Gst.PadProbeType.BUFFER, tiler_sink_pad_buffer_probe, 0)

tiler.set_property("show-source", <stream_id>) `
```

/opt/nvidia/deepstream/deepstream/sources/apps/apps-common/src/deepstream-yaml/deepstream_source_yaml.cpp有範例


## 斷線重連
rust的插件(可能可以編譯成c函式庫)
https://coaxion.net/blog/2020/07/automatic-retry-on-error-and-fallback-stream-handling-for-gstreamer-sources/  

https://gitlab.freedesktop.org/gstreamer/gst-plugins-rs/-/tree/master/utils/fallbackswitch

編譯rust插件  
https://www.collabora.com/news-and-blog/blog/2020/06/23/cross-building-rust-gstreamer-plugins-for-the-raspberry-pi/  

RUST說明書
https://rust-lang.tw/book-tw/ch01-03-hello-cargo.html  



## 截出有物件的圖
https://forums.developer.nvidia.com/t/saving-frame-with-detected-object-jetson-nano-ds4-0-2/121797/3

## 關閉Ubuntu圖形介面
https://linuxconfig.org/how-to-disable-enable-gui-on-boot-in-ubuntu-20-04-focal-fossa-linux-desktop


## 關閉使用gpu的資源
https://heary.cn/posts/Linux%E7%8E%AF%E5%A2%83%E4%B8%8B%E9%87%8D%E8%A3%85NVIDIA%E9%A9%B1%E5%8A%A8%E6%8A%A5%E9%94%99kernel-module-nvidia-modeset-in-use%E9%97%AE%E9%A2%98%E5%88%86%E6%9E%90/

發現nvidia smi persistence mode會占用GPU資源，必須釋放掉才能安裝新的driver
可以用nvidia-smi的指令關掉https://docs.nvidia.com/deploy/driver-persistence/index.html#usage
```
nvidia-smi -pm 0
```

## 移除舊的driver
```
apt-get remove --purge nvidia-driver-520
apt-get autoremove
```


參考:  
https://www.gclue.jp/2022/06/gstreamer.html  