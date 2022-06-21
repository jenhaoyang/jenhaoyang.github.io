---
title: Windows 10 build onnxruntime Tensorrt Execution Providers Windows 10 編譯onnxruntime Tensorrt Execution Providers
date: 2022-06-21 22:29:00 +0800  
categories: [深度學習工具]
tags: [Tensorrt]     # TAG names should always be lowercase
---

# 1. 安裝CUDA、cuDNN、Tensorrt
CUDA: 11.4
cuDNN: 8.2
Tensorrt:8.2.3.0

# 2. 編譯
基本上官方提供的build.bat 沒特別做什麼事，只是幫你呼叫build.py而已，所以所有參數都可以直接看build.py需要什麼，如果直接下官方文件的指令下會出錯。
```bash
build.bat --cudnn_home "C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.4" --cuda_home "C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.4" --use_tensorrt --tensorrt_home C:\TensorRT-8.2.3.0 --cuda_version 11.4 --msvc_toolset 14.11
```
