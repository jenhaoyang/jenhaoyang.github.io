---
layout: post
title: Linux下設定VScode、CMake、GCC、GDB來開發C++專案
date: 2022-10-28 17:52 +0800
---


# 安裝套件
* cmake extension
* c++ extension
![cmake extension](/assets/img/2022-08-24-12-30/vscode-cmake.png){: w="700" h="400" }
![c++ extension](/assets/img/2022-08-24-12-30/vscode-cpp.png){: w="700" h="400" }

* 安裝編譯器GCC、除錯器DBG
```bash
sudo apt-get update
sudo apt-get install build-essential gdb
gcc -v #確認GCC安裝成功
```
* 安裝CMake
直接用`sudo apt install cmake`，安裝的版本會比較舊，因此如果想用CMake最新的功能可以按照[官方網站的安裝方式](https://apt.kitware.com/)

# 編譯C++檔
用以下指令建立一個專案資料夾，最後一行`code .`會直接打開一個新的VScode並且以這個資料夾作為工作目錄
```
mkdir projects
cd projects
mkdir helloworld
cd helloworld
code .
```

在helloworld資料夾建立`helloworld.cpp`檔案並且寫入以下程式碼，然後按下編譯按鈕，並且選擇g++作為編譯器(如下圖所示)
```c
#include <iostream>
#include <vector>
#include <string>

using namespace std;

int main()
{
    vector<string> msg {"Hello", "C++", "World", "from", "VS Code", "and the C++ extension!"};

    for (const string& word : msg)
    {
        cout << word << " ";
    }
    cout << endl;
}
```

![c++ run compiler](/assets/img/2022-10-28-18-10/cpp-run-compiler.png){: w="700" h="400" }
![c++ chose compiler](/assets/img/2022-10-28-18-10/cpp-chose-compiler.png){: w="700" h="400" }


成功編譯後，你會在`Terminal`看到程式成功輸出文字
![c++ run success](/assets/img/2022-10-28-18-10/cpp-run-success.png){: w="700" h="400" }


第一次按下執行compiler後，VScode會幫你建立一個`.vscode`資料夾和一個，`tasks.json`。
在這個資料夾下可以有三種檔案，每個檔案各有自己的用處，關於`tasks.json`詳細設定可以參考[這裡](https://code.visualstudio.com/docs/editor/variables-reference)
  * tasks.json (compiler build settings)
  * launch.json (debugger settings)
  * c_cpp_properties.json (compiler path and IntelliSense settings)

在`tasks.json`裡面有幾個的比較重要的點
  *  `args`是給GCC的參數，要符合GCC參數的順序
  * 在這裡我們用`${file}`這個變數告訴GCC目前打開的檔案讓他編譯
  * `${fileDirname}`這個變數告訴GCC目前的資料夾位置，讓他在這個位置產生我們的執行檔
  * `${fileBasenameNoExtension}`變數取出目前開啟的檔名但是不包含副檔名，我們用這個名字作為我們的執行檔名，也就是helloworld
  * `label`會顯示在task清單 `detail` 會顯示在task清單的詳細描述。先按下`Ctrl+P`並且輸入`task `(task後面有空白)，就會顯示task清單，包含我們建立的task(如下圖所示)
  * 如果有多個task，可以利用`group`裡面`isDefault`屬性設定預設task

![c++ task list](/assets/img/2022-10-28-18-10/cpp-task-list.png){: w="700" h="400" }

# 除錯、設中斷點
* 在程式碼下一個中斷點，並且點及旁邊的執行按鈕並且選擇Debug就可以開始除錯了
![c++ breakpoint](/assets/img/2022-10-28-18-10/breakpoint.png){: w="700" h="400" }
![c++ run debug](/assets/img/2022-10-28-18-10/run-debug.png){: w="700" h="400" }
![c++ selece run debug](/assets/img/2022-10-28-18-10/selece-run-debug.png){: w="700" h="400" }