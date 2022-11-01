---
layout: post
title: 文章轉錄--Linux環境撰寫Shared Library
date: 2022-10-29 20:43 +0800
categories: [環境設定與部屬]
tags: [cpp]
pin: true
---

# 撰寫範例程式
```c
bool isPalindrome(char* word);
```
{: file="pal.h" }

```c
#include "pal.h"
#include <string.h>
 
bool isPalindrome(char* word)
{
    bool ret = true;
 
    char *p = word;
    int len = strlen(word);
    char *q = &word[len-1];
 
    for (int i = 0 ; i < len ; ++i, ++p, --q)
    {
        if (*p != *q)
        {
            ret = false;
        }
    }
 
    return ret;
}
```
{: file="pal.cpp" }

# 編譯shared library
在終端機輸入以下GCC指令，注意這裡我們有`-c`選項，這告訴GCC不要進行linking stage，如果沒加GCC就會報錯，因為一個應用程式一定會有main()函式，但是Library不需要main()函式。這行指令將會產生一個`pal.o`檔
```bash
g++ -fPIC -c -Wall pal.cpp
```

接下來我們要用`pal.o`檔和以下指令產生真正的library。ld是linker program，通常會被g++呼叫。`-shared`告訴ld製作一個shared object，以`pal.o`為輸入輸出名為`libpal.so`。在Linux通常shared libraries的副檔名都是.so。
```bash
ld -shared pal.o -o libpal.so
```

# 使用shared library
首先建立一個`main.cpp`來呼叫我們的library的`isPalindrome`函式。在程式中我們include函式庫的標頭檔`pal.h`
```c
#include "pal.h"
#include <iostream>
 
using namespace std;
 
int main()
{
    while (1)
    {
        char buffer[64] = {0};
        cin >> buffer;
 
        if (isPalindrome(buffer))
        {
            cout << "Word is a palindrome" << endl;
        }
        else
        {
            cout << "Word is not a palindrome" << endl;
        }
    }
 
    return 0;
}
```
{: file="main.cpp" }

接下來我們可能直接用`g++ -Wall main.cpp -lpal`這行指令連接我們的shared library，c但是如果這麼做我們會得到下面錯誤訊息。這是因為`ld`在預設的搜尋路徑下找不到`libpal.so`
```bash
/usr/bin/ld: cannot find -lpal: No such file or directory
collect2: error: ld returned 1 exit status
```
其中一種解法是用g++告入ld函式庫的位置
```bash
g++ -Wall -L<libpal.so的路徑> -Wl,-rpath=<libpal.so的路徑> main.cpp -lpal
```
例如
```bash
g++ -Wall -L/home/faye -Wl,-rpath=/home/faye/ main.cpp -lpal
```
其中:
* `-Wall`是用來檢查所有編譯警告的
* `L`式shared library的路徑，讓`ld`知道要去那裡尋找
* `-Wl`是一連串用逗號分隔的linker指令，在這裡有
  * `-rpath`表示library 的路徑會被嵌入到主程式的執行檔，因此loader在執行主程式的時候可以找到library 
 
`-L`與`-rpath`的區別是:
* `-L`是給linker用的
* `-rpath`是被嵌入到執行檔給`loader`看的

最後g++會幫我們產生a.out執行檔，執行方式和結果如下
```bash
$ ./a.out 
ada
Word is a palindrome
team
Word is not a palindrome
```

# 安裝自己開發的Shared Libraries
用剛剛的方式連接Shared Libraries的方法有個缺點，也就是你的編譯指令直接給其他人的話可能會出錯，因為Shared Libraries在每個人的電腦上的位置可能都不一樣。因此`rpath`和`-L`選項的路徑也會不一樣。而我們的解決方法之一就是`LD_LIBRARY_PATH`

## LD_LIBRARY_PATH
如果我們直接在終端機輸入指令`echo $LD_LIBRARY_PATH`看`LD_LIBRARY_PATH`的值，他應該會是空的，除非你以前曾經設定過他。要是用`LD_LIBRARY_PATH`我們只需要以下指令
```bash
export LD_LIBRARY_PATH=<Shared Library所在資料夾>:$LD_LIBRARY_PATH
```
例如
```bash
export LD_LIBRARY_PATH=/home/faye:$LD_LIBRARY_PATH
```
這時在輸入一次`echo $LD_LIBRARY_PATH`應該就會輸出`/home/faye:`。而這時候我們編譯`main.cpp`的時候就算沒有`-rpath`選項編譯出來的執行檔也不會找不到`libpal.so`
```bash
g++ -Wall -L/home/faye/sotest main.cpp -lpal
```
> 你可以先嘗試看看在還沒設定`LD_LIBRARY_PATH`之前或是利用`unset LD_LIBRARY_PATH`指令清空`LD_LIBRARY_PATH`變數，所編譯出來的執行檔會出現什麼問題。  
> 你應該會發現編譯的過程沒有任何錯誤訊息，但是一旦你執行編譯出來的執行檔`a.out`就會跳出錯誤訊息`./a.out: error while loading shared libraries: libpal.so: cannot open shared object file: No such file or directory`，這表示執行檔loader找不到`libpal.so`檔
{: .prompt-tip }

不過`LD_LIBRARY_PATH`其實只適合在開發階段拿來測試library用，因為它不需要root權限，但是函式庫發布給大家使用的時候，要求每個人都去設定`LD_LIBRARY_PATH`並不是個好方法。

## ldconfig : 安裝Shared Libraries正統方法
首先我們先清除上一個教學的`LD_LIBRARY_PATH`設定，可以用`unset LD_LIBRARY_PATH`指令來清除。  
接下來我們必須把我們的函式庫複製到`/usr/lib`資料夾。複製函式庫到`/usr/lib`必須擁有root權限，因此我們用`sudo`來幫助我們。
```bash
sudo mv <函式庫所在資料夾>/libpal.so /usr/lib
```
例如
```bash
sudo mv /home/faye/libpal.so /usr/lib
```
接下來更新系統中儲存可用libraries的cache。
```bash
sudo ldconfig
```
你可以用下面指令來確定cache已經被更新了而且系統可以找到你的函式庫
```bash
ldconfig -p | grep libpal
```
系統應該會回應你
```bash
libpal.so (libc6,x86-64) => /lib/libpal.so
```
接下來你就可以用下面指令編譯我們的執行檔了，而且這次我們不需要`-rpath`，也不需要`-L`選項!!因為現在我們的函式庫已經位於系統預設的函式庫搜尋路徑下了。
```bash
g++ -Wall main.cpp -lpal
```

參考:  
https://www.fayewilliams.com/2015/07/07/creating-a-shared-library/  
https://www.fayewilliams.com/2015/07/14/installing-and-accessing-shared-libraries/  

相關文章:  
[Linux安裝prebuild函式庫以OpenCV為例](/posts/Linux安裝prebuild函式庫以OpenCV為例/)