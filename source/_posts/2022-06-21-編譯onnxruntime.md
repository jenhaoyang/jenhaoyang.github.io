---
layout: post
title: 編譯onnxruntime Tensorrt ExecutionProvider
date: 2022-06-21 17:28 +0800
categories: [深度學習工具]
tags: [tensorrt]     # TAG names should always be lowercase
---

# 1. 安裝CUDA、cuDNN、Tensorrt
## 我的環境
* Windows10
* Visual Studio 2019(注意  先安裝visual studio 再安裝CUDA，因為CUDA包含給visual studio使用的原件)
* cmake: 3.24.0
* CUDA: 11.4
* cuDNN: 8.2
* Tensorrt:8.2.3.0

# 2.下載onnxruntime github原始碼並且切換到想要的onnxruntime 版本
```shell
git clone https://github.com/microsoft/onnxruntime.git
git checkout v1.11.1
```

# 2. 編譯
1. 不一定要機器上要插著顯卡才能編譯，沒顯卡的機器也可以編譯
2. 為了保險起見我是用系統管理員權限執行cmd，沒試過普通的cmd可不可以
3. 基本上官方提供的build.bat 沒特別做什麼事，只是幫你呼叫build.py而已，所以所有參數都可以直接看build.py需要什麼，如果直接下官方文件的指令下會出錯。
4. onnxrumtime 的github ci pipeline是很好的參考，可以用來作為build.py指令參數的參考例如[windows onnxruntime tensorrt](https://github.com/microsoft/onnxruntime/blob/4e9e01cb3c008335a2471c27dbdf7dd5d12e4224/tools/ci_build/github/azure-pipelines/win-gpu-tensorrt-ci-pipeline.yml#L80)


5. [CUDA architectures查詢](https://docs.nvidia.com/cuda/cuda-compiler-driver-nvcc/index.html#gpu-feature-list)

```shell
build.bat --parallel --config Release --build_shared_lib --build_wheel --skip_tests --cudnn_home "C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.4" --cuda_home "C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.4" --use_tensorrt --tensorrt_home C:\TensorRT-8.2.3.0 --cuda_version 11.4 --cmake_extra_defines "CMAKE_CUDA_ARCHITECTURES=80"
```
5. 如果編譯失敗請刪掉程式產生的onnxruntime\build資料夾後再重下指令

6. 從[docker file](https://github.com/microsoft/onnxruntime/blob/859ef277a0d75e90bdfc0b3f35d4f9194791aabc/dockerfiles/Dockerfile.tensorrt#L29)我們可以發現如何安裝編譯好的onnx tensorrt

```
pip install <onnxruntime 資料夾>/build/Windows/Release/Release/dist/onnxruntime_gpu-1.11.1-cp38-cp38-win_amd64.whl
```
