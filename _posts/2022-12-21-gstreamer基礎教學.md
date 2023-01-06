---
layout: post
title: GStreamer基礎教學
date: 2022-12-21 17:18 +0800
---

# GObject 和 GLib
GStreamer建立在GObject和GLib之上，熟悉GObject和GLib對於學習GStreamer會有幫助，要區分目前的函示是屬於GStreamer還是GLib的方法就是GStreamer的函式是`gst_`開頭，而GLib的函式是`g_`開頭

# 簡單範例
## 範例
下面程式碼是一個最基礎的GStreamer範例`basic-tutorial-1.c`
```c
#include <gst/gst.h>

#ifdef __APPLE__
#include <TargetConditionals.h>
#endif

int
tutorial_main (int argc, char *argv[])
{
  GstElement *pipeline;
  GstBus *bus;
  GstMessage *msg;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Build the pipeline */
  pipeline =
      gst_parse_launch
      ("playbin uri=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm",
      NULL);

  /* Start playing */
  gst_element_set_state (pipeline, GST_STATE_PLAYING);

  /* Wait until error or EOS */
  bus = gst_element_get_bus (pipeline);
  msg =
      gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE,
      GST_MESSAGE_ERROR | GST_MESSAGE_EOS);

  /* See next tutorial for proper error message handling/parsing */
  if (GST_MESSAGE_TYPE (msg) == GST_MESSAGE_ERROR) {
    g_error ("An error occurred! Re-run with the GST_DEBUG=*:WARN environment "
        "variable set for more details.");
  }

  /* Free resources */
  gst_message_unref (msg);
  gst_object_unref (bus);
  gst_element_set_state (pipeline, GST_STATE_NULL);
  gst_object_unref (pipeline);
  return 0;
}

int
main (int argc, char *argv[])
{
#if defined(__APPLE__) && TARGET_OS_MAC && !TARGET_OS_IPHONE
  return gst_macos_main (tutorial_main, argc, argv, NULL);
#else
  return tutorial_main (argc, argv);
#endif
}
```
{:file='basic-tutorial-1.c'}

在Linux下可以用以下指令編譯。
```
gcc basic-tutorial-1.c -o basic-tutorial-1 `pkg-config --cflags --libs gstreamer-1.0`
```

## 解說
首先所有的Gstreamer都必須呼叫`gst_init()`，他有三個功能
* 初始化GStreamer
* 確認plug-ins都可以使用
* 執行命令列的[參數選項](https://gstreamer.freedesktop.org/documentation/gstreamer/gst.html#gst_init)，可以直接將main函式的`argc`和`argv`直接傳入`gst_init()`
```
  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Build the pipeline */
```