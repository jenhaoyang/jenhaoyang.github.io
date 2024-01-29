---
layout: post
title: 2024-01-20-CMake編譯OpenCV-Python.md
date: 2024-01-20 17:31 +0800
---


# cmake指令編譯並且安裝
https://stackoverflow.com/a/48428846
`cmake --build . --target install`

# 安裝msvc2019 build tool
# 安裝python3.8.10

# 下載opencv-python
git clone https://github.com/opencv/opencv-python.git
cd opencv-python
git submodule update --init --recursive

# 設定GSTREAMER_ROOT_X86_64環境變數
https://gstreamer.freedesktop.org/documentation/installing/on-windows.html#building-the-tutorials

`<gstreamer資料夾>\1.0\msvc_x86_64`

# 開始編譯
pip wheel . --verbose

# GST_PLUGIN_PATH或是GST_PLUGIN_SYSTEM_PATH(不一定要加)
gstreamer預設尋找plugin的順序如下
1. %HOMEDRIVE%%HOMEFOLDER%/.gstreamer-1.0/plugins
2. C:\gstreamer\1.0\x86\lib\gstreamer-1.0
3. <location of gstreamer-1.0-0.dll>\..\lib\gstreamer-1.0
4. %GST_PLUGIN_PATH%

https://gstreamer.freedesktop.org/documentation/installing/on-windows.html#download-and-install-gstreamer-binaries
`<gstreamer資料夾>\1.0\msvc_x86_64\lib\gstreamer-1.0`

# Path變數
`<gstreamer資料夾>\1.0\msvc_x86_64\bin`


# 載入時手動加入dll路徑
os.add_dll_directory()
https://bugs.python.org/issue43173



# 測試
```python
import os
os.add_dll_directory("C:\\gstreamer\\1.0\\msvc_x86_64\\bin")
os.add_dll_directory("C:\\gstreamer\\1.0\\msvc_x86_64\\lib\\gstreamer-1.0")
import cv2
gst = 'rtspsrc location=rtsp://192.168.8.57/live.sdp ! decodebin ! videoconvert ! video/x-raw,format=BGR ! appsink drop=1'

# Variant for NVIDIA decoder that may be selected by decodebin:
# gst = 'rtspsrc location=rtsp://username:pasword@10.2.9.164:554/h264Preview_01_main latency=300 ! decodebin ! nvvidconv ! video/x-raw,format=BGRx ! videoconvert ! video/x-raw,format=BGR ! appsink drop=1'

cap = cv2.VideoCapture(gst,cv2.CAP_GSTREAMER)
while(cap.isOpened()):
  ret, frame = cap.read()
  if not ret:
    break
  cv2.imshow('frame', frame)
  if cv2.waitKey(1) & 0xFF == ord('q'):
    break

cv2.destroyAllWindows()
cap.release()
```

# gstreamer載入時錯誤檢查
1. 用 [Dependencies](https://github.com/lucasg/Dependencies) 軟體檢查dll缺少哪一些東西
2. plugin找不到的錯誤可能是找不到plugin的dll本身，或是找不到plugin所需要的dll

# 安裝CUDA、cuDNN

# CUDA 設定
從4.0之後CUDA相關的功能被移到opencv_contrib，必須要編一opencv_contrib才能啟用CUDA功能
1. WITH_CUDA勾選
2. 指定OPENCV_EXTRA_MODULES_PATH路徑




# (選擇性)G-API
G-API說明
https://github.com/opencv/opencv/wiki/Graph-API