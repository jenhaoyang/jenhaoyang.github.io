---
layout: post
title: Gstreamer使用筆記
date: 2024-10-17 17:31 +0800
---

# 關閉程式前發送EOS事件給pipeline的方法
```c
// 發送EOS訊號讓filesink可以正常結束
gst_element_send_event(appCtx[i]->pipeline.pipeline, gst_event_new_eos());
msg = gst_bus_timed_pop_filtered(GST_ELEMENT_BUS (appCtx[i]->pipeline.pipeline), 
GST_CLOCK_TIME_NONE, 
(GstMessageType) (GST_MESSAGE_ERROR | GST_MESSAGE_EOS));//gst_bus_timed_pop_filtered 是synchronously取得訊號，如果沒有訊號會一直等待並把執行續卡住
gst_element_set_state (appCtx[i]->pipeline.pipeline, GST_STATE_NULL);
g_main_loop_quit (main_loop);
```

https://gstreamer.freedesktop.org/documentation/gstreamer/gstelement.html#gst_element_get_state

https://gstreamer-devel.narkive.com/g1omJFAj/gst-devel-graceful-exit-shutdown-of-a-pipeline#post3

https://stackoverflow.com/questions/38170733/gstreamer-not-flushing-to-the-filesink

## 範例
https://gitlab.freedesktop.org/freedesktop/snippets/-/snippets/1760

https://gist.github.com/CaptainOnly/f88cbb6ac5fdeae80200fbb3f7be695c
