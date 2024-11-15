---
layout: post
title: VScode-C語言除錯技巧
date: 2024-07-15 17:31 +0800
---

# VScode

(float[5])v_anchors
查看大小為5的float array

參考:  
https://github.com/microsoft/vscode-cpptools/issues/688#issuecomment-299262784


https://stackoverflow.com/questions/52721440/how-to-expand-an-array-while-debugging-in-visual-studio-code

# GDB

列印位址
```
//變數c的位址
p&c
```

https://stackoverflow.com/questions/43861731/gdb-print-the-value-of-memory-address



列印位址的內容
```
(gdb) p *(struct node *) 0x00000000004004fc
```
https://stackoverflow.com/questions/43861731/gdb-print-the-value-of-memory-address


segmentation fault 除錯
1. gdb run 並且帶入執行參數
gdb --args executablename arg1 arg2 arg3
(gdb) run

https://stackoverflow.com/questions/6121094/how-do-i-run-a-program-with-commandline-arguments-using-gdb-within-a-bash-script

2. https://www.cnblogs.com/realjimmy/p/12850884.html
