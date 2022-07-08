---
layout: post
title: Windows下讓cmake找到OpenCV
date: 2022-07-08 16:16 +0800
categories: [環境設定與部屬]
tags: [cmake]  
---


設定完後會看到cmake輸出找到的路徑
```
Environment variable OpenCV_ROOT is set to:

    D:\lib\build_opencv
```
另外由於相容性的關係，cmake預設不會使用環境變數的OpenCV_ROOT，我們必須在CMakeLists.txt中設定打開這個功能
```
cmake_policy(SET CMP0074 NEW)
```

成功設定後可以看到cmake會有以下輸出
```
-- Found OpenCV: D:/lib/build_opencv (found version "4.6.0")
-- OpenCV library status:
--     version: 4.6.0
```

詳細寫法可以參考這個範例
https://github.com/jenhaoyang/sort-python/blob/main/CMakeLists.txt

---
參考:
https://seanzhengw.github.io/blog/cmake/2018/04/23/cmake-find-package.html  
https://cmake.org/cmake/help/v3.12/policy/CMP0074.html  
https://stackoverflow.com/questions/21314893/what-is-the-default-search-path-for-find-package-in-windows-using-cmake  