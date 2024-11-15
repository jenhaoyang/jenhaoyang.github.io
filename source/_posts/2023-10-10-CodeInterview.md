---
layout: post
title: CodeInterview
date: 2023-10-10 17:31 +0800
---



# BigO
* 時間複雜度相加還是相乘
相加
```
Add the Runtimes: 0 (A  +  B) 
1  for  (int  a  :  arrA)  { 
2      print(a); 
3  } 
4 
5  for  (int  b  :  arrB)  { 
6      print(b) ; 
7  } 
```

相乘:
```
Multiply the Runtimes: O(A* B) 
1  for  (int  a  :  arrA)  { 
2      for  (int  b  :  arrB)  { 
3          print(a  +  " , "  +  b); 
4      }     
5  } 
```

* 遞迴的時間複雜度
