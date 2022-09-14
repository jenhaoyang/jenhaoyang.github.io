---
layout: post
title: Python呼叫C++系列(一)環境設定
date: 2022-08-24 12:30 +0800
categories: [模型部屬]
tags: [pythoncpp]
---

```
我製作了一個SORT: Simple online and realtime tracking的C++的Python binding  
，歡迎參考看看  
https://github.com/jenhaoyang/sort-cpp-pybind11
```

* 作業系統: Ubuntu 20.04
* IDE: VSCode

# 安裝vscode套件
* cmake extension
* c++ extension
![cmake extension](/assets/img/2022-08-24-12-30/vscode-cmake.png){: w="700" h="400" }
![c++ extension](/assets/img/2022-08-24-12-30/vscode-cpp.png){: w="700" h="400" }


# C++ 重點提示
以下面的程式為例  
### Num的標頭檔(Num.h)
```cpp
// Num.h 的內容
class Num
{
 private:
 int num;
 public:
 Num(int n);
 int getNum();
};
```
### Num的原始碼檔(Num.cpp)
```cpp
// Num.cpp 的內容
#include "Num.h"
Num::Num() : num(0) { }
Num::Num(int n): num(n) {}
int Num::getNum()
{
 return num;
} 
```
### 主程式main的原始碼檔(main.cpp)
```cpp
// main.cpp 的內容
#include <iostream>
#include "Num.h"
using namespace std;
int main()
{
 Num n(35);
 cout << n.getNum() << endl;
 return 0;
} 
```

* ### 編譯與直譯  
Python屬於直譯式語言，而C++是屬於編譯語言。  
因為我們用文字寫的程式電腦是看不懂的，必須有個工具幫我們將程式翻譯成電腦懂的機器碼，他就像我們跟電腦之間的翻譯員。  
直譯語言用的是直譯器(Interpreter)他像是個即時翻譯員，當我們在命令提示字元或是Bash，輸入Python的時候，打開的就是直譯器，我們每輸入一行(或一段)的程式，直譯器會馬上幫我們翻譯成機器碼，所以我們可以直接看到輸出結果。  
而編譯語言使用的是編譯器(Compiler)，他向翻譯社一樣，要把所有的程式全部都寫完後，送進編譯器一口起全部翻成機器碼。
* ### 標頭檔(head file)
用來描述function或是class介面的檔案，介面的意思就是function會需要什麼參數，然後回傳直會是什麼，又或者是class有哪些method、member variable和method使用的參數和回傳值。而程式實作方式(
Implement)都放在原碼檔。就像上圖的Num.h那樣。
* ### 原碼檔(source file)
紀錄程式實作方式的檔案，所有的實作方式都會被記錄在原碼檔。
* ### 動態/靜態函式庫
有時候程式提供者不想揭露實作方式的時候，可能就會給動態/靜態函式庫以及標頭檔，因為動態/靜態函式庫是機械碼所以人類是看不懂的，要使用動態/靜態函式庫我們必須利用標頭檔來了解有哪些函式可以呼叫，並且利用標頭檔來呼叫和使用動態/靜態函式庫。

# CMake管理C++專案
如果曾經使用過Visual Studio寫過C++的話，就會看過Visual Studio幫我們製造一個.sln檔，這個檔案記錄了專案的設定。不過不同的軟體編輯器有不同的專案檔，大家常用的軟體編輯器也都不一樣。因此CMake就扮演軟體編輯器通譯的角色，只要我們可以跟CMake說我們專案的設定，CMake就能夠幫我們製造不同軟體編輯器的專案檔，而且CMake是跨平台的，因此不管開發者是在Windows還是Linux上開發，CMake都可以幫我們產生是和的專案檔，讓開發者合作順暢。

# 站在CMake之上來管理Python C++混和呼叫的專案上:scikit-build
Python C++混和的專案面臨到更多複雜的設定，單純使用CMake並不好寫，而scikit-build幫我們處理好Python C++混和型的專案常用的工具或是編譯流程，省去我們重新造輪子的工。

# 從單純CMake、C++到Python呼叫C++的混合專案

* 建立一個簡單的C++程式
利用上面C++的程式碼，在一個資料夾中把檔案都建立好，並且多一個CMakeLists.txt

* 建立一個動態函式庫

* 調整header file和source file位置


# 參考:  
https://developer.lsst.io/

下面的連結有些marco是舊的  
https://people.duke.edu/~ccc14/cspy/18G_C++_Python_pybind11.html  
https://developer.lsst.io/v/billglick-slurm-queues/coding/python_wrappers_for_cpp_with_pybind11.html