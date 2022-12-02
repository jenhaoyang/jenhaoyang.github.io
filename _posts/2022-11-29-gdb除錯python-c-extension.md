---
layout: post
title: GDB除錯Python C extension
date: 2022-11-29 16:02 +0800
---

# 下載並安裝GDB Python-debugging extension
## 下載與安裝
gdb7之後，gdb有內建一個Python interpreter，如果要查看他的版本可以輸入以下指令。  
注意!!gdb內建的Python interpreter跟我們想要除錯的Python版本沒有任何關係，他就單純只是一個拿來執行Python extension的interpreter
```
> gdb
GNU gdb (GDB) Fedora 7.10-29.fc23
Copyright (C) 2015 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
...
(gdb) python
>import sys
>print(sys.version_info)
>end
sys.version_info(major=3, minor=4, micro=3, releaselevel='final', serial=0)
(gdb)
```
接著下載[libpython.py](https://github.com/python/cpython/blob/main/Tools/gdb/libpython.py)，並且放在你任意指定的資料夾，在這裡我們放在`~/.config/gdb/libpython.py`。  
<br>
接下來建立或是修改`~/.gdbinit`把下面內容加進去。如果你是放到自訂的資料夾，記得把`~/.config/gdb`改成你的資料夾。
```
python
import gdb
import sys
import os
sys.path.insert(0, os.path.expanduser("~/.config/gdb"))
def setup_python(event):
    import libpython
gdb.events.new_objfile.connect(setup_python)
end
```
## 測試安裝成功