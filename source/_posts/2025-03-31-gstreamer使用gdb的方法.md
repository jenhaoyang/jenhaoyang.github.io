---
title: gstreamer使用gdb的方法
date: 2025-03-31 17:49:40
categories:
tags:
---

git clone https://gitlab.freedesktop.org/gstreamer/gstreamer.git
cd gstreamer/
git checkout 1.20.3

pip3 install --user meson

meson setup builddir

ninja -C builddir



測試gst環境成功
https://www.collabora.com/news-and-blog/blog/2020/03/19/getting-started-with-gstreamer-gst-build/  
1.在程式`subprojects/gst-plugins-base/gst/videotestsrc/gstvideotestsrc.c`中加料
在gst_video_test_src_start()中加上，故意讓程式報錯跳出
```c
  GST_ERROR_OBJECT (src, "Starting to debug videotestsrc, is there an error ?");
```
2.ninja -C build重新編譯程式，可以注意到這次編譯只會編譯被更新的檔案
3.執行GST_DEBUG=videotestsrc:1 gst-launch-1.0 videotestsrc num-buffers=1 ! fakevideosink，並出現錯誤，表示測試成功
```
Setting pipeline to PAUSED ...
0:00:00.225273663 21743 0x565528ab7100 ERROR           videotestsrc gstvideotestsrc.c:1216:gst_video_test_src_start: Starting to debug videotestsrc, is there an error ?
Pipeline is PREROLLING ...
Pipeline is PREROLLED ...
Setting pipeline to PLAYING ...
New clock: GstSystemClock
Got EOS from element "pipeline0".
Execution ended after 0:00:00.033464391
Setting pipeline to PAUSED ...
Setting pipeline to READY ...
Setting pipeline to NULL ...
Freeing pipeline ..
```


使用gst環境(類似python的venv)
python3 gst-env.py
以下連結說明有更新的環境變數
https://gstreamer.freedesktop.org/documentation/installing/building-from-source-using-meson.html#how-does-it-work

此時許多環境變數被自動設置，包含pkg-config也被設置，可以用pkg-config --cflags gstreamer-1.0檢查gstreamer的路徑，如此一來讓cmake編譯也可以連結到自行編譯的程式，並且包含gdb下中斷點需要的資訊


把deepstream的gst-plugin的變數加進來，這樣才找的到deepstream的plugin
export GST_PLUGIN_PATH=/opt/nvidia/deepstream/deepstream/lib/gst-plugins:$GST_PLUGIN_PATH


gdb下中斷點
gst_element_factory_make

vscode要在debug的那一個terimal執行gst-env.py才能讓中斷點停在gstreamer的程式上面