---
layout: post
title: GDB除錯Python C extension
date: 2022-11-29 16:02 +0800
---
# 測試範例
```c
#include <Python.h>

static PyObject *method_myadd(PyObject *self, PyObject *args){
    int x, y, z = -1;

    /* Parse arguments */
    if(!PyArg_ParseTuple(args, "ii", &x, &y)){
            return NULL;
    }

    /* The actual bit of code I need */
    z = x + y;

    return PyLong_FromLong(z);
}

static PyMethodDef myaddMethods[] = {
    {"myadd", method_myadd, METH_VARARGS, "Python interface for myadd C library function"},
    {NULL, NULL, 0, NULL}
};

static struct PyModuleDef myaddmodule = {
    PyModuleDef_HEAD_INIT,
    "myadd",
    "Python interface for the myadd C library function",
    -1,
    myaddMethods
};

PyMODINIT_FUNC PyInit_myadd(void) {
    return PyModule_Create(&myaddmodule);
}
```
{: file='myadd.cpp'}

```python
import myadd

print("going to ADD SOME NUMBERS")

x = myadd.myadd(5,6)

print(x)
```
{: file='myscript.py'}

```python
from distutils.core import setup, Extension

def main():
    setup(name="myadd",
          version="1.0.0",
          description="Python interface for the myadd C library function",
          author="Nadiah",
          author_email="nadiah@nadiah.org",
          ext_modules=[Extension("myadd", ["myadd.cpp"])],
          )


if __name__ == "__main__":
    main()
```
{: file='setup.py'}


# 安裝GCC
```
sudo apt-get install build-essential
```

# 安裝python標頭檔
```
apt-get install python3-dev
```

# 安裝venv
最好要使用venv來開發c extension，因為如果沒有使用venv，`python3 setup.py install`編譯好的套件
```
apt install python3-venv
```

# 編譯範例程式
```
python3 setup.py install
```

# 安裝Python debug symbol
下面指令python的版本可以改成你自己的python版本
```
apt-get install python3.10-dbg
```

# 以GDB執行Python
```
gdb python
```
注意!!必須先正確安裝Python的debug symbol再執行這一步，完成後你應該要可以看到成功載入Python debug symbol，gdb的顯示類似如下
```
GNU gdb (Ubuntu 12.1-0ubuntu1~22.04) 12.1
Copyright (C) 2022 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
.....
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from python...
Reading symbols from /usr/lib/debug/.build-id/75/c83caa11a9418b8e5ae8feb0bb8f2e5d00c47b.debug...
```
如果你看到`(No debugging symbols found in python)`表示GDB找不到Python debug symbols。



# GDB下中斷點
這一步會`預先`建立一個中斷點，引為此時我們的extension還沒被Python載入。再這個範例我們把中斷點下在`myadd.cpp`第12行`z = x + y`的位置
```
(gdb)b myadd.cpp:12
```
這時候GDB會問你是不是要建立一個未來使用的中斷點，回答是就可以
```
(gdb) b myadd.cpp:12
No source file named myadd.cpp.
Make breakpoint pending on future shared library load? (y or [n]) 
```

# Python呼叫extension
最後在GDB裡面執行`myscript.py`程式，就可以看到程式停在我們的中斷點
```
(gdb) run myscript.py
```

# debug Python 好用的指令



# 參考:  
編譯教學  
https://uwpce-pythoncert.github.io/Py300/notes/BuildingExtensions.html

安裝debug symbol  
https://wiki.python.org/moin/DebuggingWithGdb  

gdb除錯參考:  
https://scipy-lectures.org/advanced/debugging/index.html#debugging-segmentation-faults-using-gdb  


加入gdb路徑
https://devguide.python.org/advanced-tools/gdb/index.html#gdb-7-and-later

範例程式
https://nadiah.org/2020/03/01/example-debug-mixed-python-c-in-visual-studio-code/


除錯
https://developers.redhat.com/articles/2021/09/08/debugging-python-c-extensions-gdb#python_commands_in_gdb