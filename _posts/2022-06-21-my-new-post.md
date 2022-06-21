---
layout: post
title: My New Post
date: 2022-06-21 17:28 +0800
categories: [深度學習工具]
tags: [Tensorrt]     # TAG names should always be lowercase
---

# 1. 安裝CUDA、cuDNN、Tensorrt
Visual Studio 2019(注意  先安裝visual studio 再安裝CUDA，因為CUDA包含給visual studio使用的原件)
cmake: 3.24.0
CUDA: 11.4
cuDNN: 8.2
Tensorrt:8.2.3.0

# 2.下載onnxruntime github原始碼並且切換到想要的onnxruntime 版本
```
git clone https://github.com/microsoft/onnxruntime.git
git checkout v1.11.1
```

# 2. 編譯
1. 不一定要機器上要插著顯卡才能編譯，沒顯卡的機器也可以編譯
2. 為了保險起見我是用系統管理員權限執行cmd，沒試過普通的cmd可不可以
3. 基本上官方提供的build.bat 沒特別做什麼事，只是幫你呼叫build.py而已，所以所有參數都可以直接看build.py需要什麼，如果直接下官方文件的指令下會出錯。
```bash
build.bat --cudnn_home "C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.4" --cuda_home "C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.4" --use_tensorrt --tensorrt_home C:\TensorRT-8.2.3.0 --cuda_version 11.4 --msvc_toolset 14.11
```
4. 如果編譯失敗請刪掉程式產生的onnxruntime\build資料夾後再重下指令