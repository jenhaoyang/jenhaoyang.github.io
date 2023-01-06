---
layout: post
title: GStreamer基礎教學
date: 2022-12-21 17:18 +0800
---

# GObject 和 GLib
GStreamer建立在GObject和GLib之上，熟悉GObject和GLib對於學習GStreamer會有幫助，要區分目前的函示是屬於GStreamer還是GLib的方法就是GStreamer的函式是`gst_`開頭，而GLib的函式是`g_`開頭

# 1.簡單範例
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
上面這個範例最重要的只有五個需要注意的地方，其他的程式碼都是程式結束後清理的例行動作。

1. 首先所有的Gstreamer都必須呼叫`gst_init()`，他有三個功能
* 初始化GStreamer
* 確認plug-ins都可以使用
* 執行命令列的[參數選項](https://gstreamer.freedesktop.org/documentation/gstreamer/gst.html#gst_init)，可以直接將main函式的`argc`和`argv`直接傳入`gst_init()`
```
  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Build the pipeline */
```

2. gst_parse_launch
GStreamer 元件像水管一樣接起來組成像水管的`pipeline`，影音資料像像水流一樣，`source`元件是`pipeline`的起頭，像水龍頭一樣流出影音資料。`sink`元件是`pipeline`的結尾，是影音資料最後到達的地方。過程中經過中間的處理原件可以對影音資料進行處理。  

通常你會需要用程式定義每個元件和串接方式，但是如果你的pipeline很簡單，你可以直接用文字描述的方式作為參數傳給`gst_parse_launch`來建立pipeline

3. playbin
在這個範例中我們用到playbin來建立pipeline，playbin是一個特殊的元件，他可以同時做為source和sink，而且他可以自己建立成一個pipeline。在這個範例我們給他一個影片串流的URL，如果URL有錯或是指定的影片檔不存在，playbin可以回傳錯誤，在這個範例我們遇到錯誤的時候是直接離開程式。
```
      gst_parse_launch
      ("playbin uri=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm",
      NULL);
```

4. state
GStreamer 還有一個重要的觀念`state`。每一個GStreamer element都有`state`，很像影音撥放器的播放和暫停按鈕。在這個範例裡面，`pipeline`是我們唯一的element，因此要把他設為撥放才會開始撥放影片。
```
/* Start playing */
gst_element_set_state (pipeline, GST_STATE_PLAYING);
```

5. message bus、gst_element_get_bus、gst_bus_timed_pop_filtered
在下面這兩行，`gst_element_get_bus`會取得`pipeline`的bus，而`gst_bus_timed_pop_filtered`會把main thread停住直到我們感興趣的訊息，在這裡是`GST_MESSAGE_ERROR`和`GST_MESSAGE_EOS`，而`GST_MESSAGE_EOS`代表影片結束了，因此當影片結束的時候整個程式就會停止。

```
  /* Wait until error or EOS */
  bus = gst_element_get_bus (pipeline);
  msg =
      gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE,
      GST_MESSAGE_ERROR | GST_MESSAGE_EOS);
```

# 2. 