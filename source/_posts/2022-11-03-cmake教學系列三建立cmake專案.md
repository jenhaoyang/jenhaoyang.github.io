---
layout: post
title: CMake教學系列三建立CMake專案
date: 2022-11-03 16:50 +0800
categories: [環境設定與部屬]
tags: [cmake]
pin: true
---
# 下載範例
下面指令將會下載本文所需的範例
```
git clone https://gitlab.com/CLIUtils/modern-cmake.git
cd modern-cmake/examples/simple-project/
```

# CMakeLists.txt解說
* 在第29行`add_library(MyLibExample simple_lib.cpp simple_lib.hpp)`增加了一個名為`MyLibExample`的`target`並且將會編譯一個`MyLibExample`library
* 第34行增加了一個`add_executable(MyExample simple_example.cpp)`增加了一個名為`MyExample`的`target`並且將會編譯一個`MyExample`的執行檔
* 第38行`target_link_libraries(MyExample PRIVATE MyLibExample)`連接了`MyLibExample`和`MyExample`這兩個`target`，注意你必須先製作`target`才能夠連接他們

# 建置專案、編譯
接下來你可以用以下指令建置專案並且編譯。CMake會產生一個`build`資料夾，在裡面你應該會看到`libMyLibExample.a`library和`MyExample`執行檔。你可以查看`CMakeCache.txt`看有哪些`cache variable`，接下來用`./build/MyExample`執行`MyExample`執行檔
```bash
cmake -S . -B build
cmake --build build -j 8
```

# 常見錯誤
## 路徑中有空白
```
set(VAR a b v)
```
這個寫法會讓VAR變成一個list，並且擁有a b c三個元素，因此如果你的路徑像這樣，你的路徑就會被拆成兩段。
```
set(MY_DIR "/path/with spaces/")
target_include_directories(target PRIVATE ${MY_DIR})
```
解決方法就是在`${MY_DIR}`外面再包一個引號
```
set(MY_DIR "/path/with spaces/")
target_include_directories(target PRIVATE "${MY_DIR}")
```

# 除錯
## 印出變數
雖然`message`指令也可以印出變數，不過你有更好的選擇，`cmake_print_properties`、`cmake_print_variables`，記得要先`include(CMakePrintHelpers)`。如此一來你可以更加容易的印出`target`的屬性
```
include(CMakePrintHelpers)
cmake_print_variables(MY_VARIABLE)
cmake_print_properties(
    TARGETS my_target
    PROPERTIES POSITION_INDEPENDENT_CODE
)
```

## --trace-source 和 --trace-expand
`--trace-source`讓你可以指定只要看你想看的`CMakeLists.txt`，`--trace-expand`變數全部展開，例如原本是
```
add_library(simple_lib ${SOURCES} ${HEADERS} )
```
加了`--trace-expand`變成
```
add_library(simple_lib simple_lib.c simple_lib.h )
```
下載這個範例，並且試看看`cmake -S . -B build --trace-source=CMakeLists.txt` 、 `cmake -S . -B build --trace-source=CMakeLists.txt --trace-expand`有什麼不一樣
```bash
git clone git@github.com:hsf-training/hsf-training-cmake-webpage.git
cd hsf-training-cmake-webpage/code/04-debug
```

## 除錯CMake的find_...模組
沿用上面的範例，在這個範例你可以看到第16行有一個`find_library(MATH_LIBRARY m)`
```bash
git clone git@github.com:hsf-training/hsf-training-cmake-webpage.git
cd hsf-training-cmake-webpage/code/04-debug
```
試看看`cmake -S . -B build --debug-find`。記得要先清除`build`資料夾否會出現debug訊息。你也可以用[CMAKE_FIND_DEBUG_MODE](https://cmake.org/cmake/help/latest/variable/CMAKE_FIND_DEBUG_MODE.html#cmake-find-debug-mode)來針對你想要debug的find_...模組來除錯
```
set(CMAKE_FIND_DEBUG_MODE TRUE)
find_program(...)
set(CMAKE_FIND_DEBUG_MODE FALSE)
```

# 設定build types
如果你想要執行C++ debugger，你會需要設定很多flag，CMake提供四種build types來幫你設定這些flag。
* CMAKE_BUILD_TYPE=Debug : 輸出所有除錯訊息
* CMAKE_BUILD_TYPE=RelWithDebInfo : release build 不過有一些額外的除錯訊息
* CMAKE_BUILD_TYPE=Release : 最佳化release build
* CMAKE_BUILD_TYPE=MinSizeRel : minimum size release
下面範例示範如何用CMake的Debug模式編譯，並且用GDB除錯
```
cd hsf-training-cmake-webpage/code/04-debug
cmake -S . -B build-debug -DCMAKE_BUILD_TYPE=Debug
cmake --build build-debug
gdb build-debug/simple_example
```
GDB指令
```
# GDB                
break my_sin         
r       
watch sign   
c             
```

# 尋找套件
```
find_package(MyPackage 1.2)
```
這個命令會首先尋找[CMAKE_MODULE_PATH](https://cmake.org/cmake/help/latest/variable/CMAKE_MODULE_PATH.html)這份路徑清單，在這些路徑底下尋找`FindMyPackage.cmake`這個檔案，注意他是直接把`find_package`第一個參數`MyPackage`產生出`FindMyPackage.cmake`這個搜尋目標，所以如果我們寫成`find_package(OpenCV 3)`，那搜尋目標就是`FindOpenCV.cmake`。  
如果找不到`FindMyPackage.cmake`他就會接著尋找`MyPackageConfig.cmake`，如果`MyPackage_DIR`存在的話也會搜尋這個路徑。
在CMake3.12+之後，如果你的套件不是安裝在系統預設路徑，你可以設定環境變數[`<PackageName>_ROOT`](https://cmake.org/cmake/help/latest/envvar/PackageName_ROOT.html)讓CMake搜尋。以下以Bash命令設定環境變數為例
```
export HDF5_ROOT=$HOME/software/hdf5-1.12.0
```
或者是設定CMAKE環境變數，以下以Bash命令設定環境變數為例
```
export CMAKE_PREFIX_PATH=$HOME/software/hdf5-1.12.0:$HOME/software/boost-1.74.0:$CMAKE_PREFIX_PATH
```

## FindPackage.cmake
這是舊方法(`MODULE`方法)，這裡有CMake提供的[FindPackage清單](https://cmake.org/cmake/help/latest/manual/cmake-modules.7.html#find-modules)

## PackageConfig
這是由package開發者所提供，簡而言之如果你是package開發者，你應該提供`<package>Config.cmake`並且自行維護。

# CMake結合PkgConfig


參考:  
https://hsf-training.github.io/hsf-training-cmake-webpage/08-debugging/index.html  

https://hsf-training.github.io/hsf-training-cmake-webpage/09-findingpackages/index.html  

現代CMake觀念
https://pabloariasal.github.io/2018/02/19/its-time-to-do-cmake-right/  

CMake結合PkgConfig
https://stackoverflow.com/a/74038236