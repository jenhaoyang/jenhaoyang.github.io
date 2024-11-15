---
layout: post
title: Ubuntu更新cmake
date: 2022-06-28 17:31 +0800
categories: [環境設定與部屬]
tags: [cmake]
---
前一陣子在windows上編譯Nvidia triton-inference-server發現他的編譯腳本有很多方便的的操作方法，升級cmake是其中之一，注意指令中有指定ubuntu版本
cmake官方有關於apt下載的[說明](https://apt.kitware.com/)

```shell
ENV DEBIAN_FRONTEND=noninteractive
RUN apt install software-properties-common -y
RUN wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | \
      gpg --dearmor - |  \
      tee /etc/apt/trusted.gpg.d/kitware.gpg >/dev/null && \
    apt-add-repository 'deb https://apt.kitware.com/ubuntu/ focal main' && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
      cmake-data=3.21.1-0kitware1ubuntu20.04.1 cmake=3.21.1-0kitware1ubuntu20.04.1
```


---
參考:
https://github.com/triton-inference-server/server/blob/main/Dockerfile.QA#L65