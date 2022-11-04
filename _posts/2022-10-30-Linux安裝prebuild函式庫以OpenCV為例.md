---
layout: post
title: Linux安裝prebuild函式庫以OpenCV為例
date: 2022-10-30 22:02 +0800
categories: [環境設定與部屬]
tags: [cpp]
pin: true
---
> 我的部落格[文章轉錄--Linux環境撰寫Shared Library](/posts/文章轉錄-Linux環境撰寫Shared-Library)有詳細介紹如何製作和安裝Shared Library，如果想要了解更多Shared Library安裝和製作方式可以參考這篇。
{: .prompt-tip }

在這篇我們想直接使用OpenCV預編譯的函式庫，省去自己編譯函式庫的時間，首先我們先到官網提到的Third-party packages的[System packages in popular Linux distributions](https://pkgs.org/search/?q=opencv)，找到自己的Linux distributions，在這裡我們是使用Ubuntu22，而我們要下載的是`development files for opencv`也就是[libopencv-dev_4.5.4+dfsg-9ubuntu4_amd64.deb](https://ubuntu.pkgs.org/22.04/ubuntu-universe-amd64/libopencv-dev_4.5.4+dfsg-9ubuntu4_amd64.deb.html)這個連結，在`Install Howto`的地方可以看到安裝指令。
> 如果你不想弄亂你的環境，建議你可以用Vagrant建立一台測試環境來測試一下安裝後的結果，或是做一些實驗。
> 安裝Vagrant的方法在[安裝vagrant製作測試環境](/posts/安裝vagrant製作測試環境)
{: .prompt-tip }
```bash
sudo apt-get update
sudo apt-get install libopencv-dev
```
在`Files`的地方可以看到他幫我們裝了什麼東西以及他們被安裝的位置。在`Requires`的地方可以看到他還幫我們安裝了哪些相依套件。  
在這裡我們比較一下[libopencv-core-dev](https://ubuntu.pkgs.org/22.04/ubuntu-universe-amd64/libopencv-core-dev_4.5.4+dfsg-9ubuntu4_amd64.deb.html)和[libopencv-core4.5d](https://ubuntu.pkgs.org/22.04/ubuntu-universe-amd64/libopencv-core4.5d_4.5.4+dfsg-9ubuntu4_amd64.deb.html)這兩個函式庫。在這兩個package的`Files`的地方可以發現`libopencv-core-dev`幫我們在`/usr/include/opencv4`多裝了很多標頭檔(*.hpp)，因為如果要在我們自己的C++中使用OpenCV，必須要include OpenCV的標頭檔，而dev套件已經幫我們幫把標頭檔都放在`/usr/include/opencv4`讓我們可以引用了。而在編寫OpenCV的C++專案的時候，要記得把這個include資料夾放到你的專案裡。  

## pkg-config幫我們列出全部的標頭檔位置和opencv的名稱
標頭檔路徑只需要加上g++選項`-I/usr/include/opencv4`就可以了，不過如果要把所有用到的library都手動寫出來實在很麻煩，這時候`pkg-config`可以幫我們把全部的opencv library全部列出來，我們可以試看看在終端機輸入下面指令`pkg-config --libs --cflags opencv4`(如果安裝的是opencv 2.x或3.x要輸入`pkg-config --libs --cflags opencv`)，終端機的回應應該會長的像下面這樣。
```bash
$ pkg-config --libs --cflags opencv4
-I/usr/include/opencv4 -lopencv_stitching -lopencv_alphamat -lopencv_aruco -lopencv_barcode -lopencv_bgsegm -lopencv_bioinspired -lopencv_ccalib -lopencv_dnn_objdetect -lopencv_dnn_superres -lopencv_dpm -lopencv_face -lopencv_freetype -lopencv_fuzzy -lopencv_hdf -lopencv_hfs -lopencv_img_hash -lopencv_intensity_transform -lopencv_line_descriptor -lopencv_mcc -lopencv_quality -lopencv_rapid -lopencv_reg -lopencv_rgbd -lopencv_saliency -lopencv_shape -lopencv_stereo -lopencv_structured_light -lopencv_phase_unwrapping -lopencv_superres -lopencv_optflow -lopencv_surface_matching -lopencv_tracking -lopencv_highgui -lopencv_datasets -lopencv_text -lopencv_plot -lopencv_ml -lopencv_videostab -lopencv_videoio -lopencv_viz -lopencv_wechat_qrcode -lopencv_ximgproc -lopencv_video -lopencv_xobjdetect -lopencv_objdetect -lopencv_calib3d -lopencv_imgcodecs -lopencv_features2d -lopencv_dnn -lopencv_flann -lopencv_xphoto -lopencv_photo -lopencv_imgproc -lopencv_core
```

g++的指令是長這樣
```bash
/usr/bin/g++ -g main.cpp -o main `pkg-config --libs --cflags opencv4`
```

你可以到[linux下設定vscode-cmake-gcc-gdb來開發c-專案](/posts/linux下設定vscode-cmake-gcc-gdb來開發c-專案)查看如何用VSCode連接和編譯OpenCV library。

參考:  
https://blog.gtwang.org/programming/ubuntu-linux-install-opencv-cpp-python-hello-world-tutorial/  
https://pkgs.org/search/?q=opencv  
https://ubuntu.pkgs.org/22.04/ubuntu-universe-amd64/libopencv-dev_4.5.4+dfsg-9ubuntu4_amd64.deb.html  