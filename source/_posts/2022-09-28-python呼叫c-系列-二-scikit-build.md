---
layout: post
title: Python呼叫C++系列(二)scikit-build
date: 2022-09-28 17:08 +0800
categories: [模型部屬]
tags: [pythoncpp]
---
[第一篇連結](/posts/Python呼叫Cpp系列一環境設定/)
# 本篇重點
我們將建立一個可安裝的c++ extension套件，可以用pip install 安裝，本篇教學將延續第一篇的範例`example.cpp`

# 準備環境
* 安裝所需套件和scikit-build

* 安裝編譯所需套件
```bash
sudo apt install build-essential
```

* 安裝CMake
```bash
sudo apt install cmake
```

# 使用方式
* 建立一個setup.py檔並且放入下面這行  


* 新增一個CMakeLists.txt並且放入下面內容 



* 新增一個pyproject.toml並且放入下面內容 


* 測試安裝套件
pip install -v 


# Debug