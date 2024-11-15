---
layout: post
title: Ubuntu安裝CUDA cuDNN TeosorRT
date: 2023-04-27 17:28 +0800
categories: [環境設定與部屬]
tags: [ubuntu]     # TAG names should always be lowercase
---


1. 安裝CUDA
將CUDA的repo加入apt
```bash
wget https://developer.download.nvidia.com/compute/cuda/11.7.1/local_installers/cuda-repo-debian11-11-7-local_11.7.1-515.65.01-1_amd64.deb
sudo dpkg -i cuda-repo-debian11-11-7-local_11.7.1-515.65.01-1_amd64.deb
sudo rm /etc/apt/sources.list.d/*cuda*
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/3bf863cc.pub
sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/ /"
sudo apt-get update
```

>接下來注意不要直接`sudo apt-get -y install cuda`，因為可能會直接安裝最新版的CUDA，而不是你指定的版本

首先確認有哪些版本可以下載
```bash
apt-cache policy vlc
```
輸出如下
```bash
cuda:
  Installed: (none)
  Candidate: 12.1.1-1
  Version table:
     12.1.1-1 500
        500 https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64  Packages
     12.1.0-1 500
        500 https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64  Packages
     12.0.1-1 500
        500 https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64  Packages
     12.0.0-1 500
        500 https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64  Packages
     11.8.0-1 500
        500 https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64  Packages
     11.7.1-1 500
        500 https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64  Packages
     11.7.0-1 500
        500 https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64  Packages
......
```

在這裡我們想安裝CUDA 11.7.1，指令如下
```bash
sudo apt-get  install cuda=11.7.1-1
```
再次確認即將安裝的是不是CUDA11.7，是的話再按`y`

2. 安裝cuDNN和TensorRT
```bash
sudo apt-get install libnvinfer8=8.4.1-1+cuda11.6 libnvinfer-plugin8=8.4.1-1+cuda11.6 libnvparsers8=8.4.1-1+cuda11.6 \
  libnvonnxparsers8=8.4.1-1+cuda11.6 libnvinfer-bin=8.4.1-1+cuda11.6 libnvinfer-dev=8.4.1-1+cuda11.6 \
  libnvinfer-plugin-dev=8.4.1-1+cuda11.6 libnvparsers-dev=8.4.1-1+cuda11.6 libnvonnxparsers-dev=8.4.1-1+cuda11.6 \
  libnvinfer-samples=8.4.1-1+cuda11.6 libcudnn8=8.4.1.50-1+cuda11.6 libcudnn8-dev=8.4.1.50-1+cuda11.6 \
  python3-libnvinfer=8.4.1-1+cuda11.6 python3-libnvinfer-dev=8.4.1-1+cuda11.6
```



參考:
https://docs.nvidia.com/metropolis/deepstream/6.1.1/dev-guide/text/DS_Quickstart.html#  

https://askubuntu.com/questions/340530/how-can-i-check-the-available-version-of-a-package-in-the-repositories