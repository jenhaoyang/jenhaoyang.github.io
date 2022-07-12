---
layout: post
title: Visual Studio + CMake製作Python Extension
date: 2022-07-11 18:06 +0800
categories: [程式撰寫]
tags: [python_extension]  
---
版本資訊
Visual Studio 2019
CMake 3.24
Python3.8.10

## 設定Opencv
* [編譯OpenCV](https://docs.opencv.org/4.6.0/d3/d52/tutorial_windows_install.html) 
* 設定windows的環境變數 OpenCV_ROOT 到OpenCVConfig.cmake所在的資料夾(CMake 3.12之後的功能，[文件](https://cmake.org/cmake/help/latest/envvar/PackageName_ROOT.html#packagename-root)
* 從編譯好的OpenCVConfig.cmake我們可以看OpenCV提供了那些CMake變數給我們使用，可以參考[這裡](https://github.com/opencv/opencv/blob/139c44377032f58849bbab2ae454fcf14e89d762/cmake/templates/OpenCVConfig.cmake.in#L6-L43)

* [讓CMake複製dll文件](https://gist.github.com/Rod-Persky/e6b93e9ee31f9516261b)

* 設定CMake尋找Python.h

```cmake
find_package(Python 3 REQUIRED 
COMPONENTS Interpreter Development.Module NumPy) # New in cmake 3.19

include_directories(${Python_INCLUDE_DIRS})
link_directories(${Python_LIBRARY_DIRS})
message(STATUS "Python Location: ${Python_INCLUDE_DIRS}")

include_directories(${Python_NumPy_INCLUDE_DIRS})
message(STATUS "NumPy Location: ${Python_NumPy_INCLUDE_DIRS}")

find_package(OpenCV REQUIRED)
if (OpenCV_FOUND)
    # If the package has been found, several variables will
    # be set, you can find the full list with descriptions
    # in the OpenCVConfig.cmake file.
    # Print some message showing some of them
    include_directories(${OpenCV_INCLUDE_DIRS})
    link_libraries(${OpenCV_LIBRARIES})
    message(STATUS "OpenCV library status:")
    message(STATUS "    version: ${OpenCV_VERSION}")
    message(STATUS "    include path: ${OpenCV_INCLUDE_DIRS}")
else ()
    message(FATAL_ERROR "Could not locate OpenCV")
endif()
```



---
原碼參考:  


參考:  
[Python.h位置](https://docs.microsoft.com/zh-tw/visualstudio/python/working-with-c-cpp-python-in-visual-studio?view=vs-2022)  

[CMake尋找Python](https://cmake.org/cmake/help/latest/module/FindPython.html)