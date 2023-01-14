---
layout: post
title: deepstream教學
date: 2023-01-11 12:28 +0800
categories: [深度學習工具]
tags: [deepstream]     # TAG names should always be lowercase
---

# 畫出範例的結構圖
首先在`~/.bashrc`加入下面這行設定pipeline圖儲存的位置，注意GStreamer不會幫你建立資料夾，你必須確認資料夾存在
```shell
export GST_DEBUG_DUMP_DOT_DIR=/tmp
```
接下來在pipeline 狀態設為PLAYING之前加入下面這行程式
```c
GST_DEBUG_BIN_TO_DOT_FILE(pipeline, GST_DEBUG_GRAPH_SHOW_ALL, "dstest1-pipeline");
```
最後執行程式後就會產生`.dot`在前面設定的資料夾，你可以下載[Graphviz](http://www.graphviz.org/)，或是用[VScode的插件](https://marketplace.visualstudio.com/items?itemName=tintinweb.graphviz-interactive-preview)來看圖


我們以

參考:
https://gstreamer.freedesktop.org/documentation/tutorials/basic/debugging-tools.html?gi-language=c  
https://embeddedartistry.com/blog/2018/02/22/generating-gstreamer-pipeline-graphs/  