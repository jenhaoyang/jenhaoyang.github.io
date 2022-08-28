---
layout: post
title: Python呼叫C++系列(一)環境設定
date: 2022-08-24 12:30 +0800
---

* 作業系統: Ubuntu 20.04
* IDE: VSCode

# 安裝vscode套件
* cmake extension
* c++ extension
![cmake extension](/assets/img/2022-08-24-12-30/vscode-cmake.png){: w="700" h="400" }
![c++ extension](/assets/img/2022-08-24-12-30/vscode-cpp.png){: w="700" h="400" }


# C++ 重點提示
以下面的程式為例
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

```cpp
#include "Num.h"
Num::Num() : num(0) { }
Num::Num(int n): num(n) {}
int Num::getNum()
{
 return num;
} 
```

```cpp
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

* 編譯流程
* 標頭檔(head file)
* 原碼檔(ource file)
* 動態/靜態函式庫