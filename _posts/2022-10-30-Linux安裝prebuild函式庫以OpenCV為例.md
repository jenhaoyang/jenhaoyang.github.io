---
layout: post
title: Linux安裝prebuild函式庫以OpenCV為例
date: 2022-10-30 22:02 +0800
---
我的部落格[文章轉錄--Linux環境撰寫Shared Library](/posts/文章轉錄-Linux環境撰寫Shared-Library)有詳細介紹如何製作和安裝Shared Library，如果想要了解更多Shared Library安裝和製作方式可以參考這篇。

在這篇我們想直接使用OpenCV預編譯的函式庫，省去自己編譯函式庫的時間，首先我們先到官網提到的Third-party packages的[System packages in popular Linux distributions](https://pkgs.org/search/?q=opencv)，找到自己的Linux distributions，在這裡我們是使用Ubuntu22，而我們要下載的是`development files for opencv`也就是[libopencv-dev_4.5.4+dfsg-9ubuntu4_amd64.deb](https://ubuntu.pkgs.org/22.04/ubuntu-universe-amd64/libopencv-dev_4.5.4+dfsg-9ubuntu4_amd64.deb.html)這個連結，在`Install Howto`的地方可以看到安裝指令。
> 如果你不想弄亂你的環境，建議你可以用Vagrant建立一台測試環境來測試一下安裝後的結果，或是做一些實驗。
> 安裝Vagrant的方法在[安裝vagrant製作測試環境](/posts/安裝vagrant製作測試環境)
{: .prompt-tip }
```bash
sudo apt-get update
sudo apt-get install libopencv-core-dev
```
在`Files`的地方可以看到他幫我們裝了什麼東西以及他們被安裝的位置。在`Requires`的地方可以看到他還幫我們安裝了哪些相依套件，在這裡我們比較一下[libopencv-core-dev](https://ubuntu.pkgs.org/22.04/ubuntu-universe-amd64/libopencv-core-dev_4.5.4+dfsg-9ubuntu4_amd64.deb.html)和[libopencv-core4.5d](https://ubuntu.pkgs.org/22.04/ubuntu-universe-amd64/libopencv-core4.5d_4.5.4+dfsg-9ubuntu4_amd64.deb.html)這兩個函式庫。在這兩個package的`Files`的地方可以發現`libopencv-core-dev`幫我們在`/usr/include/opencv4`多裝了很多標頭檔(*.hpp)，因為如果要在我們自己的C++中使用OpenCV，必須要include OpenCV的標頭檔，而dev套件已經幫我們幫把標頭檔都放在`/usr/include/opencv4`讓我們可以引用了。而在編寫OpenCV的C++專案的時候，要記得把這個include資料夾放到你的專案裡。

### 以VScode、GCC、GDB為例撰寫一個簡單的OpenCV程式

### 以VScode、CMake為例撰寫一個簡單的OpenCV程式

參考:  
https://blog.gtwang.org/programming/ubuntu-linux-install-opencv-cpp-python-hello-world-tutorial/  
https://pkgs.org/search/?q=opencv  
https://ubuntu.pkgs.org/22.04/ubuntu-universe-amd64/libopencv-dev_4.5.4+dfsg-9ubuntu4_amd64.deb.html  