---
layout: post
title: Windows下讓cmake找到OpenCV和Eigen
date: 2022-07-08 16:16 +0800
categories: [環境設定與部屬]
tags: [cmake]  
---

## OpenCV
從(cmake 3.12)[https://cmake.org/cmake/help/v3.12/policy/CMP0074.html]之後，可以利用環境變數來告訴cmake套件的位置，在Windows下很方便，因為windows不像linux會把函式庫集中放在一起。設定方法就是利用設定環境變數
```
<PackageName>_ROOT
```
其中`<PackageName>`就是你的套件名稱，例如在這裡我們要設的環境變數就是`OpenCV_ROOT`，注意`<PackageName>`必須要和你寫在CMakeLists.txt裡面`find_package(<PackageName>)`大小寫都要一致

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
https://gist.github.com/jenhaoyang/924698b31f7e3baede14286c67d6059a

# Eigen3.4.0
使用`find_package(Eigen3 REQUIRED)`後需要`get_target_property(EIGEN3_INCLUDE_DIR Eigen3::Eigen INTERFACE_INCLUDE_DIRECTORIES)`
，根據這個issue的寫法https://gitlab.com/libeigen/eigen/-/issues/2486

---
參考:
https://seanzhengw.github.io/blog/cmake/2018/04/23/cmake-find-package.html  
https://cmake.org/cmake/help/v3.12/policy/CMP0074.html  
https://stackoverflow.com/questions/21314893/what-is-the-default-search-path-for-find-package-in-windows-using-cmake  