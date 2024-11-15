---
layout: post
title: Docker無法更新apt套件
date: 2023-08-17 17:31 +0800
---

# Docker build的時候遇到Hash Sum mismatch
解法:  
在dockerfile apt-get update之前建立一個/etc/apt/apt.conf.d/99fixbadproxy 文件如下

```dockerfile
RUN echo "Acquire::http::Pipeline-Depth 0;" >> /etc/apt/apt.conf.d/99fixbadproxy 
RUN echo "Acquire::http::No-Cache true;" >> /etc/apt/apt.conf.d/99fixbadproxy 
RUN echo "Acquire::BrokenProxy    true;" >> /etc/apt/apt.conf.d/99fixbadproxy 

RUN apt-get update
```