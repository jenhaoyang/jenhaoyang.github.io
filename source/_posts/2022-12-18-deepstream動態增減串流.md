---
layout: post
title: DeepStream動態增減串流
date: 2022-12-18 18:04 +0800
---

本篇文章參考:  
https://developer.nvidia.com/blog/managing-video-streams-in-runtime-with-the-deepstream-sdk/  
https://github.com/NVIDIA-AI-IOT/deepstream_reference_apps/tree/master/runtime_source_add_delete  

## Glib定時執行函式
為了到動態增減串流，必須在main thread之外有一個thread定期的察看目前串流的表。Glib提供了g_timeout_add_seconds這個函式讓我們可以定期呼叫函式。g_timeout_add_seconds可以讓我們設定每間隔多少時間呼叫一次函數。  
```
guint g_timeout_add_seconds (guint interval, GSourceFunc function, gpointer data)
```
`g_timeout_add_seconds`有三個參數分別是  
Interval: 每間隔多少秒呼叫函數一次  
function: 要被呼叫的函式  
data: 要傳送給函式的參數  

在我們動態增減的範例中，我們可以寫一個`watchDog`函式來讀去資料庫目前有無需要新增或刪減串流。

