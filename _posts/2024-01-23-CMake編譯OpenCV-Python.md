---
layout: post
title: 2024-01-23-CMake編譯OpenCV-Python
date: 2024-01-20 17:31 +0800
---

Here is the minimal steps.
Here is my enviornment
* Windows 10
* Visual Studio Build Tools 2019
* CMake 3.29.0-rc1 (use default install setting)
* Gstreamer:(Use default install setting)  
MSVC 64-bit (VS 2019, Release CRT)  
1.22.10 runtime installer  
1.22.10 development installer  
* Python3.8.10

For example, my gstreamer is install in D disk by default.
1. Add System Environment Variable `GSTREAMER_ROOT_X86_64` value `D:\gstreamer\1.0\msvc_x86_64`  
[gstreamer doc](https://gstreamer.freedesktop.org/documentation/installing/on-windows.html?gi-language=c#building-the-tutorials)
2. Add enviornment variable `GST_PLUGIN_PATH` value `D:\gstreamer\1.0\msvc_x86_64\lib\gstreamer-1.0`
3. Add `Path` enviornment variable value `D:\gstreamer\1.0\msvc_x86_64\bin`
4. re-login the computer
5. create a Python venv and use it
6. `pip install --verbose  --no-binary opencv-python opencv-python`
7. os.add_dll_directory() Shoud be add to code since from Python 3.8+, Python will NOT search DLL from PATH enviornment variable.[ref](https://bugs.python.org/issue43173)
8. test code  


```python
import os
os.add_dll_directory("D:\\gstreamer\\1.0\\msvc_x86_64\\bin")
import cv2
gst = 'rtspsrc location=rtsp://192.168.8.57/live1s3.sdp timeout= 30000 ! decodebin ! videoconvert ! video/x-raw,format=BGR ! appsink drop=1'

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

# 安裝cmake(不一定要裝)
# 安裝msvc2019 build tool
# 安裝python3.8.10
# 設定GSTREAMER_ROOT_X86_64環境變數
[gstreamer doc](https://gstreamer.freedesktop.org/documentation/installing/on-windows.html#building-the-tutorials)


`<gstreamer資料夾>\1.0\msvc_x86_64`
# 直接安裝source distributions版本的opencv-python(opencv 4.3.0之後的版本)
`pip install --verbose  --no-binary opencv-python opencv-python`

# GST_PLUGIN_PATH或是GST_PLUGIN_SYSTEM_PATH(不一定要加)
gstreamer預設尋找plugin的順序如下
1. %HOMEDRIVE%%HOMEFOLDER%/.gstreamer-1.0/plugins
2. C:\gstreamer\1.0\x86\lib\gstreamer-1.0
3. <location of gstreamer-1.0-0.dll>\..\lib\gstreamer-1.0
4. %GST_PLUGIN_PATH%
`<gstreamer資料夾>\1.0\msvc_x86_64\lib\gstreamer-1.0`

[gstreamer doc](https://gstreamer.freedesktop.org/documentation/installing/on-windows.html#download-and-install-gstreamer-binaries)


# Path變數
`<gstreamer資料夾>\1.0\msvc_x86_64\bin`


# 載入時手動加入dll路徑
os.add_dll_directory()
[ref](https://bugs.python.org/issue43173)



# 測試
```python
import os
os.add_dll_directory("D:\\gstreamer\\1.0\\msvc_x86_64\\bin")
import cv2
gst = 'rtspsrc location=rtsp://192.168.8.57/live1s3.sdp timeout= 30000 ! decodebin ! videoconvert ! video/x-raw,format=BGR ! appsink drop=1'

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

# 安裝CUDA

# CUDA 設定
從4.0之後CUDA相關的功能被移到opencv_contrib，必須要編一opencv_contrib才能啟用CUDA功能

1. `set CMAKE_ARGS="-DWITH_CUDA=ON"`
2. pip install --verbose  --no-binary opencv-contrib-python opencv-contrib-python




# (選擇性)G-API
G-API說明
https://github.com/opencv/opencv/wiki/Graph-API