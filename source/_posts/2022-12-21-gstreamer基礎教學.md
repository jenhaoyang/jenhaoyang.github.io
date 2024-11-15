---
layout: post
title: GStreamer基礎教學
date: 2022-12-21 17:18 +0800
categories: [深度學習工具]
tags: [gstreamer]
pin: true
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
```c
  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Build the pipeline */
```

2. gst_parse_launch
GStreamer 元件像水管一樣接起來組成像水管的`pipeline`，影音資料像像水流一樣，`source`元件是`pipeline`的起頭，像水龍頭一樣流出影音資料。`sink`元件是`pipeline`的結尾，是影音資料最後到達的地方。過程中經過中間的處理原件可以對影音資料進行處理。  

通常你會需要用程式定義每個元件和串接方式，但是如果你的pipeline很簡單，你可以直接用文字描述的方式作為參數傳給`gst_parse_launch`來建立pipeline

3. playbin
在這個範例中我們用到playbin來建立pipeline，playbin是一個特殊的元件，他可以同時做為source和sink，而且他可以自己建立成一個pipeline。在這個範例我們給他一個影片串流的URL，如果URL有錯或是指定的影片檔不存在，playbin可以回傳錯誤，在這個範例我們遇到錯誤的時候是直接離開程式。
```c
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

```c
  /* Wait until error or EOS */
  bus = gst_element_get_bus (pipeline);
  msg =
      gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE,
      GST_MESSAGE_ERROR | GST_MESSAGE_EOS);
```

# 2. GStreamer 觀念
這個教學將會示範用程式建立pipeline。在這裡將會學到
* 介紹GStreamer element並且學習如何建立
* 串接element
* 客製化element行為
* 利用message bus監看錯誤是件並且從中取出錯誤訊息
  
## 用程式寫出前一個教學的撥放器
完整程式碼`basic-tutorial-2.c`
```c
#include <gst/gst.h>

#ifdef __APPLE__
#include <TargetConditionals.h>
#endif

int
tutorial_main (int argc, char *argv[])
{
  GstElement *pipeline, *source, *sink;
  GstBus *bus;
  GstMessage *msg;
  GstStateChangeReturn ret;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Create the elements */
  source = gst_element_factory_make ("videotestsrc", "source");
  sink = gst_element_factory_make ("autovideosink", "sink");

  /* Create the empty pipeline */
  pipeline = gst_pipeline_new ("test-pipeline");

  if (!pipeline || !source || !sink) {
    g_printerr ("Not all elements could be created.\n");
    return -1;
  }

  /* Build the pipeline */
  gst_bin_add_many (GST_BIN (pipeline), source, sink, NULL);
  if (gst_element_link (source, sink) != TRUE) {
    g_printerr ("Elements could not be linked.\n");
    gst_object_unref (pipeline);
    return -1;
  }

  /* Modify the source's properties */
  g_object_set (source, "pattern", 0, NULL);

  /* Start playing */
  ret = gst_element_set_state (pipeline, GST_STATE_PLAYING);
  if (ret == GST_STATE_CHANGE_FAILURE) {
    g_printerr ("Unable to set the pipeline to the playing state.\n");
    gst_object_unref (pipeline);
    return -1;
  }

  /* Wait until error or EOS */
  bus = gst_element_get_bus (pipeline);
  msg =
      gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE,
      GST_MESSAGE_ERROR | GST_MESSAGE_EOS);

  /* Parse message */
  if (msg != NULL) {
    GError *err;
    gchar *debug_info;

    switch (GST_MESSAGE_TYPE (msg)) {
      case GST_MESSAGE_ERROR:
        gst_message_parse_error (msg, &err, &debug_info);
        g_printerr ("Error received from element %s: %s\n",
            GST_OBJECT_NAME (msg->src), err->message);
        g_printerr ("Debugging information: %s\n",
            debug_info ? debug_info : "none");
        g_clear_error (&err);
        g_free (debug_info);
        break;
      case GST_MESSAGE_EOS:
        g_print ("End-Of-Stream reached.\n");
        break;
      default:
        /* We should not reach here because we only asked for ERRORs and EOS */
        g_printerr ("Unexpected message received.\n");
        break;
    }
    gst_message_unref (msg);
  }

  /* Free resources */
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
{:file='basic-tutorial-2.c'}

## GStreamer element
element是GStreamer最根本的元素，影音資料從`source`流向`sink`，過程中經過`filter`對影音資料進行處理，這三個元素組成為`pipeline`  

![pipeline](/assets/img/2023-01-06-16-33/figure-1.png)



## 建立element
```
/* Create the elements */
source = gst_element_factory_make ("videotestsrc", "source");
sink = gst_element_factory_make ("autovideosink", "sink");
```
建立element可以用`gst_element_factory_make()`來建立，第一個參數是要建立的element名稱，第二個參數是我們給element取的名字。幫element命名的好處是如果你沒有儲存pointer，你可以用名稱來找到這個element，而且除錯訊息也會變得比較有意義。

這個教學中建立兩個element，`videotestsrc`和`autovideosink`，然後沒有建立任何`filter`。所以pipeline長的像下圖。  

![basic pipeline](/assets/img/2023-01-06-16-33/basic-concepts-pipeline.png)

[videotestsrc](https://gstreamer.freedesktop.org/documentation/videotestsrc/index.html#videotestsrc)是一個source element，他會產生除錯用的影像。

[autovideosink](https://gstreamer.freedesktop.org/documentation/autodetect/autovideosink.html#autovideosink)是一個sink element，他會將影像顯示到螢幕上。

## 建立pipeline
```c
/* Create the empty pipeline */
pipeline = gst_pipeline_new ("test-pipeline");

if (!pipeline || !source || !sink) {
  g_printerr ("Not all elements could be created.\n");
  return -1;
}
```
所有的element都必須在pipeline裡面才能運作，利用`gst_pipeline_new()`，可以建立新的pipeline。  

## bin
```c
 /* Build the pipeline */
  gst_bin_add_many (GST_BIN (pipeline), source, sink, NULL);
  if (gst_element_link (source, sink) != TRUE) {
    g_printerr ("Elements could not be linked.\n");
    gst_object_unref (pipeline);
    return -1;
  }
```
Gstreamer的[bin](https://gstreamer.freedesktop.org/documentation/gstreamer/gstbin.html?gi-language=c#GstBin)是也是一個element，它可以拿來成裝其他 GstElement。GstBin的類別關係圖如下  
```
GObject
    ╰──GInitiallyUnowned
        ╰──GstObject
            ╰──GstElement
                ╰──GstBin
```

pipeline也是[bin](https://gstreamer.freedesktop.org/documentation/gstreamer/gstbin.html?gi-language=c#GstBin)，用來裝入element。  

利用`gst_bin_add_many()`可以將element放入pipeline，他可以一次加入許多element。他的第一個參數是`bin`，第二個參數之後都是element，也就是要放入的element

## 連接element
到目前為止element並還沒有連接起來，利用`gst_element_link()`將element聯接起來。他的第一個參數是來源，第二個參數是目標，也就是第一個參數的element會把資料傳給第二個參數的element，所以是有順序性的。  

注意，只有位於同一個bin的element可以互相聯接。

## 屬性
GStreamer 的element全部都是一種特殊的[GObject](https://docs.gtk.org/gobject/#GObject-struct)，因此他們都有`property`，有些可以讀取有些可以寫入

`property`必須透過[GLib](https://docs.gtk.org/gobject/#g-object-get)的方法`g_object_get()`和`g_object_set()`來讀取和寫入，因此注意到這輛個函式是`g_`開頭。

`g_object_set()`支援一次修改多個`properties`。

回到我們的程式，我們改變videotestsrc的"pattern" properties，你可以將它改成[其他類型](https://gstreamer.freedesktop.org/documentation/videotestsrc/index.html#GstVideoTestSrcPattern)來看看輸出的畫面有什麼改變。
```c
/* Modify the source's properties */
g_object_set (source, "pattern", 0, NULL);
```

## 錯誤檢查
到目前為止pipeline都已經建立完成，接下來我們要增加一些程式來應付錯誤發生的情況。
```c
  /* Start playing */
  ret = gst_element_set_state (pipeline, GST_STATE_PLAYING);
  if (ret == GST_STATE_CHANGE_FAILURE) {
    g_printerr ("Unable to set the pipeline to the playing state.\n");
    gst_object_unref (pipeline);
    return -1;
  }
```
這是我們一樣呼叫`gst_element_set_state`來開始撥放，但是我們另外檢查`gst_element_set_state`回傳的錯誤。

另外，我們還多了一些程式來處理`gst_bus_timed_pop_filtered()`拿到的錯誤訊息，錯誤訊息是一個[GstMessage](https://gstreamer.freedesktop.org/documentation/gstreamer/gstmessage.html#GstMessage)，如果遇到[EOS](https://gstreamer.freedesktop.org/documentation/gstreamer/gstmessage.html#GST_MESSAGE_EOS)以外的錯誤訊息，就把他印出來。

[GstMessage](https://gstreamer.freedesktop.org/documentation/gstreamer/gstmessage.html#GstMessage)十分方便，幾乎可以承載任何形式的訊息，而GSstreamer提供了許多解析訊息的函式。

首先我們先利用[GST_MESSAGE_TYPE()](https://gstreamer.freedesktop.org/documentation/gstreamer/gstmessage.html#GST_MESSAGE_TYPE)來取得錯誤的類型，如果不是EOS錯誤，就再用`gst_message_parse_error()`把錯誤轉型成GLib的[GError](https://docs.gtk.org/glib/#GError)以及錯誤訊息的文字。注意使用完之後要記得釋放。
```
  /* Parse message */
  if (msg != NULL) {
    GError *err;
    gchar *debug_info;

    switch (GST_MESSAGE_TYPE (msg)) {
      case GST_MESSAGE_ERROR:
        gst_message_parse_error (msg, &err, &debug_info);
        g_printerr ("Error received from element %s: %s\n",
            GST_OBJECT_NAME (msg->src), err->message);
        g_printerr ("Debugging information: %s\n",
            debug_info ? debug_info : "none");
        g_clear_error (&err);
        g_free (debug_info);
        break;
      case GST_MESSAGE_EOS:
        g_print ("End-Of-Stream reached.\n");
        break;
      default:
        /* We should not reach here because we only asked for ERRORs and EOS */
        g_printerr ("Unexpected message received.\n");
        break;
    }
    gst_message_unref (msg);
  }
```

## GStreamer bus
GStreamer bus是用來將元素所產生的GstMessage按順序傳送給應用程式的thread。這裡要強調的是，消息是從處理影像的thread傳遞給應用程式的thread。

訊息可以用`gst_bus_timed_pop_filtered()`這個函式來同步取得。或是用signals 來非同步取得，應用設必須隨時注意bus上有沒有出現錯誤。

# 動態pipeline
在這個範例所建立的pipeline並不是完整的，而這裡將示範如何在程式運行時才將完整的pipeline建立好。

程式將打開一個multiplexed (或是 muxed)的檔案，也就是聲音和影像都儲存在一個檔案(container file)。用來打開這種檔案的element稱為`demuxers`。常見的container file有mp4、MKV、WMV等等。

如果container file裡面包含多個串流(例如一個影片串流，兩個聲音串流)，demuxer就會把他們分別分配到不同的出口，而不同的pipeline就可以各自處理這些串流。

在GStreamer裡element用來傳遞資料的接口稱為`pad`(GstPad)，`sink pad`就是資料流進element的口，以及`source pad`就是element將資料流出的口。記憶的方式就是`source elements`只會擁有`source pad`，而`sink element`只會擁有`sink pad`。`filter element`則同時擁有`source pad`和`sink pad`。

![src](/assets/img/2023-01-06-16-33/src-element.png)

![sink](/assets/img/2023-01-06-16-33/sink-element.png)

![filter](/assets/img/2023-01-06-16-33/filter-element.png)

在這個範例裡面，demuxer包含一個`sink pad` 和多個 `source pad`，而demuxer複雜的地方就在於在讀取檔案之前沒辦法確定demuxer到底有多少個`source pad`，因為demuxer要讀取到檔案之後才能決定有多少個`source pad`。

如此一來，demuxers一開始是沒有任何`source pad`的，因此也沒辦法在編譯時就將demuxers跟其他element連接。當程式開始運作後並且讀取到檔案時，這時候才是連接`demuxer`的時機。

為了簡單起見這個範例只連接audio pad而忽略video pad。

## 範例
下面範例`basic-tutorial-3.c`將示範動態pipeline
```c
#include <gst/gst.h>

/* Structure to contain all our information, so we can pass it to callbacks */
typedef struct _CustomData {
  GstElement *pipeline;
  GstElement *source;
  GstElement *convert;
  GstElement *resample;
  GstElement *sink;
} CustomData;

/* Handler for the pad-added signal */
static void pad_added_handler (GstElement *src, GstPad *pad, CustomData *data);

int main(int argc, char *argv[]) {
  CustomData data;
  GstBus *bus;
  GstMessage *msg;
  GstStateChangeReturn ret;
  gboolean terminate = FALSE;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Create the elements */
  data.source = gst_element_factory_make ("uridecodebin", "source");
  data.convert = gst_element_factory_make ("audioconvert", "convert");
  data.resample = gst_element_factory_make ("audioresample", "resample");
  data.sink = gst_element_factory_make ("autoaudiosink", "sink");

  /* Create the empty pipeline */
  data.pipeline = gst_pipeline_new ("test-pipeline");

  if (!data.pipeline || !data.source || !data.convert || !data.resample || !data.sink) {
    g_printerr ("Not all elements could be created.\n");
    return -1;
  }

  /* Build the pipeline. Note that we are NOT linking the source at this
   * point. We will do it later. */
  gst_bin_add_many (GST_BIN (data.pipeline), data.source, data.convert, data.resample, data.sink, NULL);
  if (!gst_element_link_many (data.convert, data.resample, data.sink, NULL)) {
    g_printerr ("Elements could not be linked.\n");
    gst_object_unref (data.pipeline);
    return -1;
  }

  /* Set the URI to play */
  g_object_set (data.source, "uri", "https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm", NULL);

  /* Connect to the pad-added signal */
  g_signal_connect (data.source, "pad-added", G_CALLBACK (pad_added_handler), &data);

  /* Start playing */
  ret = gst_element_set_state (data.pipeline, GST_STATE_PLAYING);
  if (ret == GST_STATE_CHANGE_FAILURE) {
    g_printerr ("Unable to set the pipeline to the playing state.\n");
    gst_object_unref (data.pipeline);
    return -1;
  }

  /* Listen to the bus */
  bus = gst_element_get_bus (data.pipeline);
  do {
    msg = gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE,
        GST_MESSAGE_STATE_CHANGED | GST_MESSAGE_ERROR | GST_MESSAGE_EOS);

    /* Parse message */
    if (msg != NULL) {
      GError *err;
      gchar *debug_info;

      switch (GST_MESSAGE_TYPE (msg)) {
        case GST_MESSAGE_ERROR:
          gst_message_parse_error (msg, &err, &debug_info);
          g_printerr ("Error received from element %s: %s\n", GST_OBJECT_NAME (msg->src), err->message);
          g_printerr ("Debugging information: %s\n", debug_info ? debug_info : "none");
          g_clear_error (&err);
          g_free (debug_info);
          terminate = TRUE;
          break;
        case GST_MESSAGE_EOS:
          g_print ("End-Of-Stream reached.\n");
          terminate = TRUE;
          break;
        case GST_MESSAGE_STATE_CHANGED:
          /* We are only interested in state-changed messages from the pipeline */
          if (GST_MESSAGE_SRC (msg) == GST_OBJECT (data.pipeline)) {
            GstState old_state, new_state, pending_state;
            gst_message_parse_state_changed (msg, &old_state, &new_state, &pending_state);
            g_print ("Pipeline state changed from %s to %s:\n",
                gst_element_state_get_name (old_state), gst_element_state_get_name (new_state));
          }
          break;
        default:
          /* We should not reach here */
          g_printerr ("Unexpected message received.\n");
          break;
      }
      gst_message_unref (msg);
    }
  } while (!terminate);

  /* Free resources */
  gst_object_unref (bus);
  gst_element_set_state (data.pipeline, GST_STATE_NULL);
  gst_object_unref (data.pipeline);
  return 0;
}

/* This function will be called by the pad-added signal */
static void pad_added_handler (GstElement *src, GstPad *new_pad, CustomData *data) {
  GstPad *sink_pad = gst_element_get_static_pad (data->convert, "sink");
  GstPadLinkReturn ret;
  GstCaps *new_pad_caps = NULL;
  GstStructure *new_pad_struct = NULL;
  const gchar *new_pad_type = NULL;

  g_print ("Received new pad '%s' from '%s':\n", GST_PAD_NAME (new_pad), GST_ELEMENT_NAME (src));

  /* If our converter is already linked, we have nothing to do here */
  if (gst_pad_is_linked (sink_pad)) {
    g_print ("We are already linked. Ignoring.\n");
    goto exit;
  }

  /* Check the new pad's type */
  new_pad_caps = gst_pad_get_current_caps (new_pad);
  new_pad_struct = gst_caps_get_structure (new_pad_caps, 0);
  new_pad_type = gst_structure_get_name (new_pad_struct);
  if (!g_str_has_prefix (new_pad_type, "audio/x-raw")) {
    g_print ("It has type '%s' which is not raw audio. Ignoring.\n", new_pad_type);
    goto exit;
  }

  /* Attempt the link */
  ret = gst_pad_link (new_pad, sink_pad);
  if (GST_PAD_LINK_FAILED (ret)) {
    g_print ("Type is '%s' but link failed.\n", new_pad_type);
  } else {
    g_print ("Link succeeded (type '%s').\n", new_pad_type);
  }

exit:
  /* Unreference the new pad's caps, if we got them */
  if (new_pad_caps != NULL)
    gst_caps_unref (new_pad_caps);

  /* Unreference the sink pad */
  gst_object_unref (sink_pad);
}
```
{:file='basic-tutorial-3.c'}

## 解說
首先我們先將資料組成一個struct以便後面使用
```
/* Structure to contain all our information, so we can pass it to callbacks */
typedef struct _CustomData {
  GstElement *pipeline;
  GstElement *source;
  GstElement *convert;
  GstElement *resample;
  GstElement *sink;
} CustomData;
```
接下來這行是`forward reference`晚一點會實做這個函式。
```
/* Handler for the pad-added signal */
static void pad_added_handler (GstElement *src, GstPad *pad, CustomData *data);
```
接下來建立element，在這裡`uridecodebin`會自動初始化需要的element(sources, demuxers and decoders)以便將URI轉換成影音串流。跟`playbin`比起來他只完成了一半，因為它包含了`demuxers`，所以只有到執行階段的時候`source pad`才會被初始化。

`audioconvert`用來轉換audio的格式。`audioresample`用來調整audio的sample rate。

`autoaudiosink`和`autovideosink`類似，他將會把聲音串流輸出到音效卡
```
/* Create the elements */
data.source = gst_element_factory_make ("uridecodebin", "source");
data.convert = gst_element_factory_make ("audioconvert", "convert");
data.resample = gst_element_factory_make ("audioresample", "resample");
data.sink = gst_element_factory_make ("autoaudiosink", "sink");
```

## 串接element
接下來我們將converter, resample and sink這些element連接起來。注意這時候還不可以連接source，因為這時候souce還沒有source pad。  
```c
if (!gst_element_link_many (data.convert, data.resample, data.sink, NULL)) {
  g_printerr ("Elements could not be linked.\n");
  gst_object_unref (data.pipeline);
  return -1;
}
```
然後設定source要讀取的URI
```
/* Set the URI to play */
g_object_set (data.source, "uri", "https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm", NULL);
```

## Signals
`GSignals`是GStreamer的一個重點，他讓我們可以在我們感興趣的事情發生的時候通知我們。GStreamer用名稱來區分signal，而每一個`GObject`也都有自己的signal。
``
在這個範例我們將會關心source(也就是uridecodebin element)發出來的`pad-added`這個訊號。我們必須用`g_signal_connect()`來連接訊號並且給他callback function(pad_added_handler)和我們的data pointer，讓callback functiony在號發生的時候執行。

GStreamer不會對data pointer做任何事情，他只是單純的把data pointer傳進我們的callback function，以便我們可以傳送參數給callback function。  
```c
/* Connect to the pad-added signal */
g_signal_connect (data.source, "pad-added", G_CALLBACK (pad_added_handler), &data);
```

在這個範例我們傳入字定義的data struct `CustomData`。

## 我們的callback function
當source element有足夠的資訊可以產生source pad的時候，就會觸發"pad-added"訊號，而這時候我們的callback就會被呼叫。
```c
static void pad_added_handler (GstElement *src, GstPad *new_pad, CustomData *data)
```
在我們的callback中，第一個參數是觸發訊號的`GstElement`，也就是`uridecodebin`。  

第二個參數是source剛剛產生的pad，也就是我們想要連接的pad。  

第三個參數是一個pointer，我們將用他來傳入我麼的參數給callback。

在callback裡面，我們將CustomData裡的converter element，利用`gst_element_get_static_pad ()`將他的sink pad取出來。他也就是要跟source新產生的pad對接的pad。
```
GstPad *sink_pad = gst_element_get_static_pad (data->convert, "sink");
```
在上一個範例我們讓GStreamer自己決定要連接的pad，在這裡我們將手動連接pad。首先加入下面這段程式碼以免pad重複被連接
```c
/* If our converter is already linked, we have nothing to do here */
if (gst_pad_is_linked (sink_pad)) {
  g_print ("We are already linked. Ignoring.\n");
  goto exit;
}
```
接下來我們檢查新產生的pad他產生的資料是什麼，因為我們只需要連接audio而忽略video。而且我們不能把video的pad和audio的pad對接。

`gst_pad_get_current_caps()`可以查到pad會輸出什麼資料，pad的"能力"(capabilities)被紀錄在`GstCaps`裡面。而pad所有可用的能力可以用`gst_pad_query_caps()`查詢

GstCaps 裡面可能包含許多的GstStructure，每一個都代表不同的"能力"。

由於目前我們知道新產生的pad只會有一個capabilities，所以我們直接用`gst_caps_get_structure()`取得他的第一個GstStructure。

最後再利用`gst_structure_get_name()`來取得這個GstStructure的名稱，這裡將會有關於pad傳出來的資料格式。

假如我們拿到的pad輸出的資料格式不是`audio/x-raw`，那就不是我們要的pad。如果是的話我們就連接他。

用`gst_element_link()`可以直接連接兩個pad，參數的順序是source在來sink，而且這兩個pad所在的element必須要在同一個bin裡面才可以連接。如此一來我們就完成了。
```c
/* Attempt the link */
ret = gst_pad_link (new_pad, sink_pad);
if (GST_PAD_LINK_FAILED (ret)) {
  g_print ("Type is '%s' but link failed.\n", new_pad_type);
} else {
  g_print ("Link succeeded (type '%s').\n", new_pad_type);
}
```

## GStreamer States
GStreamer共有四種狀態
* NULL: 無狀態或初始狀態
* READY: element已經準備好進入PAUSED狀態
* PAUSED: element已經暫停，並且準備好處理資料。sink element這時候只接受一個buffer，之後就阻塞
* PLAYING: element正在撥放，clock正在運作，資料正在傳輸。

注意，你只能從移動到鄰近的狀態。也就是不可以直接從NUL跳到PLAYING。當你設定pipeline 為PLAYING的時候，GSstreamer自動幫你處理這些事情。

下面這段程式監聽message bus，每當狀態有改變的時候就印出來讓你知道。每一個element都會丟出目前狀態的訊息，所以我們過濾出pipeline的狀態訊息。

```
case GST_MESSAGE_STATE_CHANGED:
  /* We are only interested in state-changed messages from the pipeline */
  if (GST_MESSAGE_SRC (msg) == GST_OBJECT (data.pipeline)) {
    GstState old_state, new_state, pending_state;
    gst_message_parse_state_changed (msg, &old_state, &new_state, &pending_state);
    g_print ("Pipeline state changed from %s to %s:\n",
        gst_element_state_get_name (old_state), gst_element_state_get_name (new_state));
  }
  break;
```

# 時間管理
在這節將會學習到GStreamer時間相關的功能包含
* 向pipeline詢問資訊例如目前串流的位置和長度。
* 尋找(跳躍)到串流上不同的位置(時間)

## GstQuery
`GstQuery`是一個用來詢問element或是pad的資訊的機制。在這個範例我們將詢問pipeline是否可以可以搜尋時間(例如如果是串流就沒辦法搜尋時間)，如果可以我們就可以在影片時間軸上跳躍。

這這個範例我們每隔一段時間就向pipeline詢問目前的影片時間位置，如此一來我們就可以將時間顯示在我們的螢幕上。

## 範例
我們將以範例`basic-tutorial-4.c`為例。
```c
#include <gst/gst.h>

/* Structure to contain all our information, so we can pass it around */
typedef struct _CustomData {
  GstElement *playbin;  /* Our one and only element */
  gboolean playing;      /* Are we in the PLAYING state? */
  gboolean terminate;    /* Should we terminate execution? */
  gboolean seek_enabled; /* Is seeking enabled for this media? */
  gboolean seek_done;    /* Have we performed the seek already? */
  gint64 duration;       /* How long does this media last, in nanoseconds */
} CustomData;

/* Forward definition of the message processing function */
static void handle_message (CustomData *data, GstMessage *msg);

int main(int argc, char *argv[]) {
  CustomData data;
  GstBus *bus;
  GstMessage *msg;
  GstStateChangeReturn ret;

  data.playing = FALSE;
  data.terminate = FALSE;
  data.seek_enabled = FALSE;
  data.seek_done = FALSE;
  data.duration = GST_CLOCK_TIME_NONE;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Create the elements */
  data.playbin = gst_element_factory_make ("playbin", "playbin");

  if (!data.playbin) {
    g_printerr ("Not all elements could be created.\n");
    return -1;
  }

  /* Set the URI to play */
  g_object_set (data.playbin, "uri", "https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm", NULL);

  /* Start playing */
  ret = gst_element_set_state (data.playbin, GST_STATE_PLAYING);
  if (ret == GST_STATE_CHANGE_FAILURE) {
    g_printerr ("Unable to set the pipeline to the playing state.\n");
    gst_object_unref (data.playbin);
    return -1;
  }

  /* Listen to the bus */
  bus = gst_element_get_bus (data.playbin);
  do {
    msg = gst_bus_timed_pop_filtered (bus, 100 * GST_MSECOND,
        GST_MESSAGE_STATE_CHANGED | GST_MESSAGE_ERROR | GST_MESSAGE_EOS | GST_MESSAGE_DURATION);

    /* Parse message */
    if (msg != NULL) {
      handle_message (&data, msg);
    } else {
      /* We got no message, this means the timeout expired */
      if (data.playing) {
        gint64 current = -1;

        /* Query the current position of the stream */
        if (!gst_element_query_position (data.playbin, GST_FORMAT_TIME, &current)) {
          g_printerr ("Could not query current position.\n");
        }

        /* If we didn't know it yet, query the stream duration */
        if (!GST_CLOCK_TIME_IS_VALID (data.duration)) {
          if (!gst_element_query_duration (data.playbin, GST_FORMAT_TIME, &data.duration)) {
            g_printerr ("Could not query current duration.\n");
          }
        }

        /* Print current position and total duration */
        g_print ("Position %" GST_TIME_FORMAT " / %" GST_TIME_FORMAT "\r",
            GST_TIME_ARGS (current), GST_TIME_ARGS (data.duration));

        /* If seeking is enabled, we have not done it yet, and the time is right, seek */
        if (data.seek_enabled && !data.seek_done && current > 10 * GST_SECOND) {
          g_print ("\nReached 10s, performing seek...\n");
          gst_element_seek_simple (data.playbin, GST_FORMAT_TIME,
              GST_SEEK_FLAG_FLUSH | GST_SEEK_FLAG_KEY_UNIT, 30 * GST_SECOND);
          data.seek_done = TRUE;
        }
      }
    }
  } while (!data.terminate);

  /* Free resources */
  gst_object_unref (bus);
  gst_element_set_state (data.playbin, GST_STATE_NULL);
  gst_object_unref (data.playbin);
  return 0;
}

static void handle_message (CustomData *data, GstMessage *msg) {
  GError *err;
  gchar *debug_info;

  switch (GST_MESSAGE_TYPE (msg)) {
    case GST_MESSAGE_ERROR:
      gst_message_parse_error (msg, &err, &debug_info);
      g_printerr ("Error received from element %s: %s\n", GST_OBJECT_NAME (msg->src), err->message);
      g_printerr ("Debugging information: %s\n", debug_info ? debug_info : "none");
      g_clear_error (&err);
      g_free (debug_info);
      data->terminate = TRUE;
      break;
    case GST_MESSAGE_EOS:
      g_print ("End-Of-Stream reached.\n");
      data->terminate = TRUE;
      break;
    case GST_MESSAGE_DURATION:
      /* The duration has changed, mark the current one as invalid */
      data->duration = GST_CLOCK_TIME_NONE;
      break;
    case GST_MESSAGE_STATE_CHANGED: {
      GstState old_state, new_state, pending_state;
      gst_message_parse_state_changed (msg, &old_state, &new_state, &pending_state);
      if (GST_MESSAGE_SRC (msg) == GST_OBJECT (data->playbin)) {
        g_print ("Pipeline state changed from %s to %s:\n",
            gst_element_state_get_name (old_state), gst_element_state_get_name (new_state));

        /* Remember whether we are in the PLAYING state or not */
        data->playing = (new_state == GST_STATE_PLAYING);

        if (data->playing) {
          /* We just moved to PLAYING. Check if seeking is possible */
          GstQuery *query;
          gint64 start, end;
          query = gst_query_new_seeking (GST_FORMAT_TIME);
          if (gst_element_query (data->playbin, query)) {
            gst_query_parse_seeking (query, NULL, &data->seek_enabled, &start, &end);
            if (data->seek_enabled) {
              g_print ("Seeking is ENABLED from %" GST_TIME_FORMAT " to %" GST_TIME_FORMAT "\n",
                  GST_TIME_ARGS (start), GST_TIME_ARGS (end));
            } else {
              g_print ("Seeking is DISABLED for this stream.\n");
            }
          }
          else {
            g_printerr ("Seeking query failed.");
          }
          gst_query_unref (query);
        }
      }
    } break;
    default:
      /* We should not reach here */
      g_printerr ("Unexpected message received.\n");
      break;
  }
  gst_message_unref (msg);
}
```

## 定義資料struct
```c
/* Structure to contain all our information, so we can pass it around */
typedef struct _CustomData {
  GstElement *playbin;  /* Our one and only element */
  gboolean playing;      /* Are we in the PLAYING state? */
  gboolean terminate;    /* Should we terminate execution? */
  gboolean seek_enabled; /* Is seeking enabled for this media? */
  gboolean seek_done;    /* Have we performed the seek already? */
  gint64 duration;       /* How long does this media last, in nanoseconds */
} CustomData;

/* Forward definition of the message processing function */
static void handle_message (CustomData *data, GstMessage *msg);
```
首先定義這個範例會用的到資料struct，同時我們也定義一個handle_message來處理我們的資料。

## 設定監聽消息的timeout
這個範例我們將會設定`gst_bus_timed_pop_filtered()`的timeout，如果0.1秒鐘內沒有收到任何訊息，就會發出`gst_bus_timed_pop_filtered()`會回傳一個NULL。timeoout的設定必須用到`GstClockTime`，因此我們直接用GST_SECOND 或 GST_MSECOND來產生`GstClockTime`時間。
```c
msg = gst_bus_timed_pop_filtered (bus, 100 * GST_MSECOND,
    GST_MESSAGE_STATE_CHANGED | GST_MESSAGE_ERROR | GST_MESSAGE_EOS | GST_MESSAGE_DURATION);
```

## 更新使用者介面
首先檢查pipeline是PLAYING的才對pipeline做查詢以免出錯。
```c
/* We got no message, this means the timeout expired */
if (data.playing) {
```
接著用`GstElement`提供的方法取得時間。
```c
/* Query the current position of the stream */
if (!gst_element_query_position (data.pipeline, GST_FORMAT_TIME, &current)) {
  g_printerr ("Could not query current position.\n");
}
``` 
如果無法取得就改成檢查是否可以詢問stream的長度
```c
/* If we didn't know it yet, query the stream duration */
if (!GST_CLOCK_TIME_IS_VALID (data.duration)) {
  if (!gst_element_query_duration (data.pipeline, GST_FORMAT_TIME, &data.duration)) {
     g_printerr ("Could not query current duration.\n");
  }
}
```
接下來就可以詢問影片長度
```c
/* Print current position and total duration */
g_print ("Position %" GST_TIME_FORMAT " / %" GST_TIME_FORMAT "\r",
    GST_TIME_ARGS (current), GST_TIME_ARGS (data.duration));
```
下一段是在影片時間軸跳躍的程式，利用`gst_element_seek_simple()`來達成。
```c
/* If seeking is enabled, we have not done it yet, and the time is right, seek */
if (data.seek_enabled && !data.seek_done && current > 10 * GST_SECOND) {
  g_print ("\nReached 10s, performing seek...\n");
  gst_element_seek_simple (data.pipeline, GST_FORMAT_TIME,
      GST_SEEK_FLAG_FLUSH | GST_SEEK_FLAG_KEY_UNIT, 30 * GST_SECOND);
  data.seek_done = TRUE;
}
```
1. GST_FORMAT_TIME: 目標時間的格式
2. GstSeekFlags: 指令跳躍的行為
  * GST_SEEK_FLAG_FLUSH: 直接拋棄掉目標時間之前的所有畫面。
  * GST_SEEK_FLAG_KEY_UNIT: 移動到目標時間附近的key frame
  * GST_SEEK_FLAG_ACCURATE: 精準的移動到目標時間上。
3. 目標時間: 是指定要跳躍到的時間位置

## Message Pump
首先如果影片長度改變我們就先讓pipeline不能被詢問影片時間。
```c
case GST_MESSAGE_DURATION:
  /* The duration has changed, mark the current one as invalid */
  data->duration = GST_CLOCK_TIME_NONE;
  break;
```
接下來這段程式如果pipeline狀態改變，確認pipeline為PAUSED或是PLAYING才可以在時間軸跳躍
```c
    case GST_MESSAGE_STATE_CHANGED: {
      GstState old_state, new_state, pending_state;
      gst_message_parse_state_changed (msg, &old_state, &new_state, &pending_state);
      if (GST_MESSAGE_SRC (msg) == GST_OBJECT (data->playbin)) {
        g_print ("Pipeline state changed from %s to %s:\n",
            gst_element_state_get_name (old_state), gst_element_state_get_name (new_state));

        /* Remember whether we are in the PLAYING state or not */
        data->playing = (new_state == GST_STATE_PLAYING);

        if (data->playing) {
          /* We just moved to PLAYING. Check if seeking is possible */
          GstQuery *query;
          gint64 start, end;
          query = gst_query_new_seeking (GST_FORMAT_TIME);
          if (gst_element_query (data->playbin, query)) {
            gst_query_parse_seeking (query, NULL, &data->seek_enabled, &start, &end);
            if (data->seek_enabled) {
              g_print ("Seeking is ENABLED from %" GST_TIME_FORMAT " to %" GST_TIME_FORMAT "\n",
                  GST_TIME_ARGS (start), GST_TIME_ARGS (end));
            } else {
              g_print ("Seeking is DISABLED for this stream.\n");
            }
          }
          else {
            g_printerr ("Seeking query failed.");
          }
          gst_query_unref (query);
        }
      }
    }
```
`gst_query_new_seeking()`建立一個新的query物件，這個query物件利用`gst_element_query()`函式被發送到pipeline。如果咬讀去回傳結果可以用`gst_query_parse_seeking()`

# 媒體格式和pad Capabilities
## Pads
Capabilities 顯示在這個Pad上流動的資料的模樣。例如他可能是"解析度300x200的RGB影片，FPS 30"  

Pad可以有多種Capabilities，例如一個video sink可以同時支援RGB和YUV格式。

Capabilities也可以是一個範圍，例如audio sink可以支援samples rates 1~48000。

如果要將兩個element連接起來，他們必須要擁有共同的Capabilities。

如果連接的時候Capabilities沒辦法對應，就會出現`negotiation error`

## Pad templates
Pad 是從Pad templates產生的，他可以產生各種Capabilities 的Pad。

## Capabilities範例
下面是一個sink pad。他支援整數型態的raw audio，包含unsigned 8-bit或是 signed, 16-bit little endian。`[]`裡面代表範圍例如channels可能是一個或兩個。
```
SINK template: 'sink'
  Availability: Always
  Capabilities:
    audio/x-raw
               format: S16LE
                 rate: [ 1, 2147483647 ]
             channels: [ 1, 2 ]
    audio/x-raw
               format: U8
                 rate: [ 1, 2147483647 ]
             channels: [ 1, 2 ]
```

注意有些Capabilities跟平台是有相關性的，要直到READY state的時候才能確定能不能用。

## 範例
`basic-tutorial-6.c`
```c
#include <gst/gst.h>

/* Functions below print the Capabilities in a human-friendly format */
static gboolean print_field (GQuark field, const GValue * value, gpointer pfx) {
  gchar *str = gst_value_serialize (value);

  g_print ("%s  %15s: %s\n", (gchar *) pfx, g_quark_to_string (field), str);
  g_free (str);
  return TRUE;
}

static void print_caps (const GstCaps * caps, const gchar * pfx) {
  guint i;

  g_return_if_fail (caps != NULL);

  if (gst_caps_is_any (caps)) {
    g_print ("%sANY\n", pfx);
    return;
  }
  if (gst_caps_is_empty (caps)) {
    g_print ("%sEMPTY\n", pfx);
    return;
  }

  for (i = 0; i < gst_caps_get_size (caps); i++) {
    GstStructure *structure = gst_caps_get_structure (caps, i);

    g_print ("%s%s\n", pfx, gst_structure_get_name (structure));
    gst_structure_foreach (structure, print_field, (gpointer) pfx);
  }
}

/* Prints information about a Pad Template, including its Capabilities */
static void print_pad_templates_information (GstElementFactory * factory) {
  const GList *pads;
  GstStaticPadTemplate *padtemplate;

  g_print ("Pad Templates for %s:\n", gst_element_factory_get_longname (factory));
  if (!gst_element_factory_get_num_pad_templates (factory)) {
    g_print ("  none\n");
    return;
  }

  pads = gst_element_factory_get_static_pad_templates (factory);
  while (pads) {
    padtemplate = pads->data;
    pads = g_list_next (pads);

    if (padtemplate->direction == GST_PAD_SRC)
      g_print ("  SRC template: '%s'\n", padtemplate->name_template);
    else if (padtemplate->direction == GST_PAD_SINK)
      g_print ("  SINK template: '%s'\n", padtemplate->name_template);
    else
      g_print ("  UNKNOWN!!! template: '%s'\n", padtemplate->name_template);

    if (padtemplate->presence == GST_PAD_ALWAYS)
      g_print ("    Availability: Always\n");
    else if (padtemplate->presence == GST_PAD_SOMETIMES)
      g_print ("    Availability: Sometimes\n");
    else if (padtemplate->presence == GST_PAD_REQUEST) {
      g_print ("    Availability: On request\n");
    } else
      g_print ("    Availability: UNKNOWN!!!\n");

    if (padtemplate->static_caps.string) {
      GstCaps *caps;
      g_print ("    Capabilities:\n");
      caps = gst_static_caps_get (&padtemplate->static_caps);
      print_caps (caps, "      ");
      gst_caps_unref (caps);

    }

    g_print ("\n");
  }
}

/* Shows the CURRENT capabilities of the requested pad in the given element */
static void print_pad_capabilities (GstElement *element, gchar *pad_name) {
  GstPad *pad = NULL;
  GstCaps *caps = NULL;

  /* Retrieve pad */
  pad = gst_element_get_static_pad (element, pad_name);
  if (!pad) {
    g_printerr ("Could not retrieve pad '%s'\n", pad_name);
    return;
  }

  /* Retrieve negotiated caps (or acceptable caps if negotiation is not finished yet) */
  caps = gst_pad_get_current_caps (pad);
  if (!caps)
    caps = gst_pad_query_caps (pad, NULL);

  /* Print and free */
  g_print ("Caps for the %s pad:\n", pad_name);
  print_caps (caps, "      ");
  gst_caps_unref (caps);
  gst_object_unref (pad);
}

int main(int argc, char *argv[]) {
  GstElement *pipeline, *source, *sink;
  GstElementFactory *source_factory, *sink_factory;
  GstBus *bus;
  GstMessage *msg;
  GstStateChangeReturn ret;
  gboolean terminate = FALSE;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Create the element factories */
  source_factory = gst_element_factory_find ("audiotestsrc");
  sink_factory = gst_element_factory_find ("autoaudiosink");
  if (!source_factory || !sink_factory) {
    g_printerr ("Not all element factories could be created.\n");
    return -1;
  }

  /* Print information about the pad templates of these factories */
  print_pad_templates_information (source_factory);
  print_pad_templates_information (sink_factory);

  /* Ask the factories to instantiate actual elements */
  source = gst_element_factory_create (source_factory, "source");
  sink = gst_element_factory_create (sink_factory, "sink");

  /* Create the empty pipeline */
  pipeline = gst_pipeline_new ("test-pipeline");

  if (!pipeline || !source || !sink) {
    g_printerr ("Not all elements could be created.\n");
    return -1;
  }

  /* Build the pipeline */
  gst_bin_add_many (GST_BIN (pipeline), source, sink, NULL);
  if (gst_element_link (source, sink) != TRUE) {
    g_printerr ("Elements could not be linked.\n");
    gst_object_unref (pipeline);
    return -1;
  }

  /* Print initial negotiated caps (in NULL state) */
  g_print ("In NULL state:\n");
  print_pad_capabilities (sink, "sink");

  /* Start playing */
  ret = gst_element_set_state (pipeline, GST_STATE_PLAYING);
  if (ret == GST_STATE_CHANGE_FAILURE) {
    g_printerr ("Unable to set the pipeline to the playing state (check the bus for error messages).\n");
  }

  /* Wait until error, EOS or State Change */
  bus = gst_element_get_bus (pipeline);
  do {
    msg = gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE, GST_MESSAGE_ERROR | GST_MESSAGE_EOS |
        GST_MESSAGE_STATE_CHANGED);

    /* Parse message */
    if (msg != NULL) {
      GError *err;
      gchar *debug_info;

      switch (GST_MESSAGE_TYPE (msg)) {
        case GST_MESSAGE_ERROR:
          gst_message_parse_error (msg, &err, &debug_info);
          g_printerr ("Error received from element %s: %s\n", GST_OBJECT_NAME (msg->src), err->message);
          g_printerr ("Debugging information: %s\n", debug_info ? debug_info : "none");
          g_clear_error (&err);
          g_free (debug_info);
          terminate = TRUE;
          break;
        case GST_MESSAGE_EOS:
          g_print ("End-Of-Stream reached.\n");
          terminate = TRUE;
          break;
        case GST_MESSAGE_STATE_CHANGED:
          /* We are only interested in state-changed messages from the pipeline */
          if (GST_MESSAGE_SRC (msg) == GST_OBJECT (pipeline)) {
            GstState old_state, new_state, pending_state;
            gst_message_parse_state_changed (msg, &old_state, &new_state, &pending_state);
            g_print ("\nPipeline state changed from %s to %s:\n",
                gst_element_state_get_name (old_state), gst_element_state_get_name (new_state));
            /* Print the current capabilities of the sink element */
            print_pad_capabilities (sink, "sink");
          }
          break;
        default:
          /* We should not reach here because we only asked for ERRORs, EOS and STATE_CHANGED */
          g_printerr ("Unexpected message received.\n");
          break;
      }
      gst_message_unref (msg);
    }
  } while (!terminate);

  /* Free resources */
  gst_object_unref (bus);
  gst_element_set_state (pipeline, GST_STATE_NULL);
  gst_object_unref (pipeline);
  gst_object_unref (source_factory);
  gst_object_unref (sink_factory);
  return 0;
}
```
## 印出capabilities 
`print_field`、`print_caps`和`print_pad_templates`可以印出capabilities。


`gst_element_get_static_pad()`可以用名稱取得Pad，之所以static是因為這個pad永遠都會出現在這個element裡面。

`gst_pad_get_current_caps()`可以取得Pad目前的capabilities，這個capabilities是固定的也可能之後會改變，甚只有可能目前沒有capabilities，要看negotiation proces的狀態來決定。

我們可以用`gst_pad_query_caps()`來取得在NULL state時的Capabilities 

## GstElementFactory
`GstElementFactory`用來初始化指定的element。

`gst_element_factory_make()` = `gst_element_factory_find()`+ `gst_element_factory_create()`

## gst_pad_get_current_caps() 和 gst_pad_query_caps() 的差別
* gst_pad_get_current_caps() : 目前可用的Capabilities
* gst_pad_query_caps() : 包含所有可能的Capabilities

# 多執行續和Pad Availability
通常GStreamer會自己處理多執行續，但有時候會需要手動處理他。

## Multithreading
GStreamer 是一個Multithreading的框架。他會自己產生和消滅thread，甚至plugin可以在自己的process裡面產生新的thread，例如video decoder會自己產生四個thread。

## Multithreading範例
下面是一個多執行續的pipeline，通常多個sink的pipeline是Multithreading
![Alt text](/assets/img/2023-01-06-16-33/basic-tutorial-7.png)

## Request pads
在前面的範例我們已經知道`uridecodebin`在執行時才會確定產生多少個pad，這種pad稱為`Sometimes Pads`，而固定不變的pad稱為`Always Pads`。

第三中Pad稱為`Request Pad`，是有需要的時候才建立。在`tee`裡面就有`Request Pad`。你必須主動建立才會產生`Request Pad`。`Request Pad`沒辦法自動連接，必須手動連接。

另外如果在`PLAYING`或`PAUSED`的時候建立或是釋放`Request Pad`要特別小心，因為有時候會造成(Pad blocking)。通常在`NULL` 或 `READY`狀態的時候建立或釋放比較安全。

## 範例
`basic-tutorial-7.c`
```c
#include <gst/gst.h>

int main(int argc, char *argv[]) {
  GstElement *pipeline, *audio_source, *tee, *audio_queue, *audio_convert, *audio_resample, *audio_sink;
  GstElement *video_queue, *visual, *video_convert, *video_sink;
  GstBus *bus;
  GstMessage *msg;
  GstPad *tee_audio_pad, *tee_video_pad;
  GstPad *queue_audio_pad, *queue_video_pad;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Create the elements */
  audio_source = gst_element_factory_make ("audiotestsrc", "audio_source");
  tee = gst_element_factory_make ("tee", "tee");
  audio_queue = gst_element_factory_make ("queue", "audio_queue");
  audio_convert = gst_element_factory_make ("audioconvert", "audio_convert");
  audio_resample = gst_element_factory_make ("audioresample", "audio_resample");
  audio_sink = gst_element_factory_make ("autoaudiosink", "audio_sink");
  video_queue = gst_element_factory_make ("queue", "video_queue");
  visual = gst_element_factory_make ("wavescope", "visual");
  video_convert = gst_element_factory_make ("videoconvert", "csp");
  video_sink = gst_element_factory_make ("autovideosink", "video_sink");

  /* Create the empty pipeline */
  pipeline = gst_pipeline_new ("test-pipeline");

  if (!pipeline || !audio_source || !tee || !audio_queue || !audio_convert || !audio_resample || !audio_sink ||
      !video_queue || !visual || !video_convert || !video_sink) {
    g_printerr ("Not all elements could be created.\n");
    return -1;
  }

  /* Configure elements */
  g_object_set (audio_source, "freq", 215.0f, NULL);
  g_object_set (visual, "shader", 0, "style", 1, NULL);

  /* Link all elements that can be automatically linked because they have "Always" pads */
  gst_bin_add_many (GST_BIN (pipeline), audio_source, tee, audio_queue, audio_convert, audio_resample, audio_sink,
      video_queue, visual, video_convert, video_sink, NULL);
  if (gst_element_link_many (audio_source, tee, NULL) != TRUE ||
      gst_element_link_many (audio_queue, audio_convert, audio_resample, audio_sink, NULL) != TRUE ||
      gst_element_link_many (video_queue, visual, video_convert, video_sink, NULL) != TRUE) {
    g_printerr ("Elements could not be linked.\n");
    gst_object_unref (pipeline);
    return -1;
  }

  /* Manually link the Tee, which has "Request" pads */
  tee_audio_pad = gst_element_get_request_pad (tee, "src_%u");
  g_print ("Obtained request pad %s for audio branch.\n", gst_pad_get_name (tee_audio_pad));
  queue_audio_pad = gst_element_get_static_pad (audio_queue, "sink");
  tee_video_pad = gst_element_get_request_pad (tee, "src_%u");
  g_print ("Obtained request pad %s for video branch.\n", gst_pad_get_name (tee_video_pad));
  queue_video_pad = gst_element_get_static_pad (video_queue, "sink");
  if (gst_pad_link (tee_audio_pad, queue_audio_pad) != GST_PAD_LINK_OK ||
      gst_pad_link (tee_video_pad, queue_video_pad) != GST_PAD_LINK_OK) {
    g_printerr ("Tee could not be linked.\n");
    gst_object_unref (pipeline);
    return -1;
  }
  gst_object_unref (queue_audio_pad);
  gst_object_unref (queue_video_pad);

  /* Start playing the pipeline */
  gst_element_set_state (pipeline, GST_STATE_PLAYING);

  /* Wait until error or EOS */
  bus = gst_element_get_bus (pipeline);
  msg = gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE, GST_MESSAGE_ERROR | GST_MESSAGE_EOS);

  /* Release the request pads from the Tee, and unref them */
  gst_element_release_request_pad (tee, tee_audio_pad);
  gst_element_release_request_pad (tee, tee_video_pad);
  gst_object_unref (tee_audio_pad);
  gst_object_unref (tee_video_pad);

  /* Free resources */
  if (msg != NULL)
    gst_message_unref (msg);
  gst_object_unref (bus);
  gst_element_set_state (pipeline, GST_STATE_NULL);

  gst_object_unref (pipeline);
  return 0;
}
```

## 初始化element
```c
/* Create the elements */
audio_source = gst_element_factory_make ("audiotestsrc", "audio_source");
tee = gst_element_factory_make ("tee", "tee");
audio_queue = gst_element_factory_make ("queue", "audio_queue");
audio_convert = gst_element_factory_make ("audioconvert", "audio_convert");
  audio_resample = gst_element_factory_make ("audioresample", "audio_resample");
audio_sink = gst_element_factory_make ("autoaudiosink", "audio_sink");
video_queue = gst_element_factory_make ("queue", "video_queue");
visual = gst_element_factory_make ("wavescope", "visual");
video_convert = gst_element_factory_make ("videoconvert", "video_convert");
video_sink = gst_element_factory_make ("autovideosink", "video_sink");
```

## 調整element
為了範例需求，微調屬性
```c
/* Configure elements */
g_object_set (audio_source, "freq", 215.0f, NULL);
g_object_set (visual, "shader", 0, "style", 1, NULL);
```

## 將element放進pipeline
將所有element放入pipeline並且連接所有`Always Pads`
```c
/* Link all elements that can be automatically linked because they have "Always" pads */
gst_bin_add_many (GST_BIN (pipeline), audio_source, tee, audio_queue, audio_convert, audio_sink,
    video_queue, visual, video_convert, video_sink, NULL);
if (gst_element_link_many (audio_source, tee, NULL) != TRUE ||
    gst_element_link_many (audio_queue, audio_convert, audio_sink, NULL) != TRUE ||
    gst_element_link_many (video_queue, visual, video_convert, video_sink, NULL) != TRUE) {
  g_printerr ("Elements could not be linked.\n");
  gst_object_unref (pipeline);
  return -1;
}
```
注意:  
`gst_element_link_many()`其實也可以自動連接`Request Pads`。因為它會自動請求新的pad。但是問題是你還是必須要手動釋放`Request Pads`。所以最好的做法是手動連接`Request Pads`。

## 連接Request Pads
要連接`Request Pads`必須要先跟element請求，而因為element有時候可以產生不同的request pads，所以要提供所需的Pad Template名稱。

在tee的文件中可以看到他有兩個Pad Template，一個是"sink"，另一個是"src_%u"(也就是Request pads)，我們可以用`gst_element_request_pad_simple()`函式來請求兩個Pads(一個給audio一個給video)。

```
Pad Templates:
  SRC template: 'src_%u'
    Availability: On request
    Capabilities:
      ANY
  
  SINK template: 'sink'
    Availability: Always
    Capabilities:
      ANY
```

接著我們需要取得下游queue element的Always pad，因此用`gst_element_get_static_pad()`。

我們用`gst_pad_link()`連接Pads。`gst_pad_link()`內部其實就是`gst_element_link()`和`gst_element_link_many()`

我們用來儲存下游queue always pad的變數`queue_audio_pad`和`queue_video_pad`要記得釋放，以免佔用reference count。
```c
/* Manually link the Tee, which has "Request" pads */
tee_audio_pad = gst_element_get_request_pad (tee, "src_%u");
g_print ("Obtained request pad %s for audio branch.\n", gst_pad_get_name (tee_audio_pad));
queue_audio_pad = gst_element_get_static_pad (audio_queue, "sink");
tee_video_pad = gst_element_get_request_pad (tee, "src_%u");
g_print ("Obtained request pad %s for video branch.\n", gst_pad_get_name (tee_video_pad));
queue_video_pad = gst_element_get_static_pad (video_queue, "sink");
if (gst_pad_link (tee_audio_pad, queue_audio_pad) != GST_PAD_LINK_OK ||
    gst_pad_link (tee_video_pad, queue_video_pad) != GST_PAD_LINK_OK) {
  g_printerr ("Tee could not be linked.\n");
  gst_object_unref (pipeline);
  return -1;
}
gst_object_unref (queue_audio_pad);
gst_object_unref (queue_video_pad);
```

最後在程式結束後，要記得釋放request pad
```c
/* Release the request pads from the Tee, and unref them */
gst_element_release_request_pad (tee, tee_audio_pad);
gst_element_release_request_pad (tee, tee_video_pad);
gst_object_unref (tee_audio_pad);
gst_object_unref (tee_video_pad);
```
`gst_element_release_request_pad()`從tee釋放requests pads，`gst_object_unref`釋放`tee_audio_pad`變數

# hort-cutting the pipeline Goal
pipeline的資料並不是封閉的，我們可以從外界注入資料給pipeline，也可以從pipeline內取得資料

## appsrc、appsink
把資料注入pipeline的element為`appsrc`，相反的從pipeline取得資料的element是`appsink`。這裡sink和source的概念是從GStreamer應用程式的角度來看，你可以想像`appsrc`也是一個source，只不過他的資料來源是來自於應用程式，相反的`appsink`就像普通的`sink`只是他最後流向應用程式。

appsrc有多種模式，在`pull`模式每當需要的時候他將會向應用程式索取資料。在`push`模式則是應用程式主動推送資料進去。在`push`模式中應用程式還可以阻塞push function當已經推入足夠多的資料到pipeline裡面的時候，或者他可以監聽`enough-data`和`need-data`訊號。

## Buffers
數據以Chunks方式進入pipeline的方式稱為Buffers，Buffers代表一單位的資料，但是每一個Buffers大小不一定一樣大。注意，我們不應該假設一個Buffers進入element就同時會有一個離開element。element可以隨意地讓Buffers停留在element內部。

Source pad產生Buffers，而sink pad接收Buffers，Gstreamer將這些Buffers一流過每一個element。

## GstBuffer 和 GstMemory 
GstBuffers 可能包含一個或一個以上的memory buffer，而真正的記憶體buffer被抽像化為`GstMemory`，因此一個`GstBuffer`可以包含一個或一個以上的`GstMemory`

每一個buffer都有時間戳和長度，以及他需要被decode的長度。

## 範例
下面範例延續Multithreading and Pad Availability的範例並擴展。

首先`audiotestsrc`被置換成`appsrc`來產生audio資料。

第二個是增加一個新的`tee`分支，這隻分支會接著`appsink`，`appsink`會將資訊回傳給應用程式。

範例`basic-tutorial-8.c`的程式如下
```c
#include <gst/gst.h>
#include <gst/audio/audio.h>
#include <string.h>

#define CHUNK_SIZE 1024   /* Amount of bytes we are sending in each buffer */
#define SAMPLE_RATE 44100 /* Samples per second we are sending */

/* Structure to contain all our information, so we can pass it to callbacks */
typedef struct _CustomData {
  GstElement *pipeline, *app_source, *tee, *audio_queue, *audio_convert1, *audio_resample, *audio_sink;
  GstElement *video_queue, *audio_convert2, *visual, *video_convert, *video_sink;
  GstElement *app_queue, *app_sink;

  guint64 num_samples;   /* Number of samples generated so far (for timestamp generation) */
  gfloat a, b, c, d;     /* For waveform generation */

  guint sourceid;        /* To control the GSource */

  GMainLoop *main_loop;  /* GLib's Main Loop */
} CustomData;

/* This method is called by the idle GSource in the mainloop, to feed CHUNK_SIZE bytes into appsrc.
 * The idle handler is added to the mainloop when appsrc requests us to start sending data (need-data signal)
 * and is removed when appsrc has enough data (enough-data signal).
 */
static gboolean push_data (CustomData *data) {
  GstBuffer *buffer;
  GstFlowReturn ret;
  int i;
  GstMapInfo map;
  gint16 *raw;
  gint num_samples = CHUNK_SIZE / 2; /* Because each sample is 16 bits */
  gfloat freq;

  /* Create a new empty buffer */
  buffer = gst_buffer_new_and_alloc (CHUNK_SIZE);

  /* Set its timestamp and duration */
  GST_BUFFER_TIMESTAMP (buffer) = gst_util_uint64_scale (data->num_samples, GST_SECOND, SAMPLE_RATE);
  GST_BUFFER_DURATION (buffer) = gst_util_uint64_scale (num_samples, GST_SECOND, SAMPLE_RATE);

  /* Generate some psychodelic waveforms */
  gst_buffer_map (buffer, &map, GST_MAP_WRITE);
  raw = (gint16 *)map.data;
  data->c += data->d;
  data->d -= data->c / 1000;
  freq = 1100 + 1000 * data->d;
  for (i = 0; i < num_samples; i++) {
    data->a += data->b;
    data->b -= data->a / freq;
    raw[i] = (gint16)(500 * data->a);
  }
  gst_buffer_unmap (buffer, &map);
  data->num_samples += num_samples;

  /* Push the buffer into the appsrc */
  g_signal_emit_by_name (data->app_source, "push-buffer", buffer, &ret);

  /* Free the buffer now that we are done with it */
  gst_buffer_unref (buffer);

  if (ret != GST_FLOW_OK) {
    /* We got some error, stop sending data */
    return FALSE;
  }

  return TRUE;
}

/* This signal callback triggers when appsrc needs data. Here, we add an idle handler
 * to the mainloop to start pushing data into the appsrc */
static void start_feed (GstElement *source, guint size, CustomData *data) {
  if (data->sourceid == 0) {
    g_print ("Start feeding\n");
    data->sourceid = g_idle_add ((GSourceFunc) push_data, data);
  }
}

/* This callback triggers when appsrc has enough data and we can stop sending.
 * We remove the idle handler from the mainloop */
static void stop_feed (GstElement *source, CustomData *data) {
  if (data->sourceid != 0) {
    g_print ("Stop feeding\n");
    g_source_remove (data->sourceid);
    data->sourceid = 0;
  }
}

/* The appsink has received a buffer */
static GstFlowReturn new_sample (GstElement *sink, CustomData *data) {
  GstSample *sample;

  /* Retrieve the buffer */
  g_signal_emit_by_name (sink, "pull-sample", &sample);
  if (sample) {
    /* The only thing we do in this example is print a * to indicate a received buffer */
    g_print ("*");
    gst_sample_unref (sample);
    return GST_FLOW_OK;
  }

  return GST_FLOW_ERROR;
}

/* This function is called when an error message is posted on the bus */
static void error_cb (GstBus *bus, GstMessage *msg, CustomData *data) {
  GError *err;
  gchar *debug_info;

  /* Print error details on the screen */
  gst_message_parse_error (msg, &err, &debug_info);
  g_printerr ("Error received from element %s: %s\n", GST_OBJECT_NAME (msg->src), err->message);
  g_printerr ("Debugging information: %s\n", debug_info ? debug_info : "none");
  g_clear_error (&err);
  g_free (debug_info);

  g_main_loop_quit (data->main_loop);
}

int main(int argc, char *argv[]) {
  CustomData data;
  GstPad *tee_audio_pad, *tee_video_pad, *tee_app_pad;
  GstPad *queue_audio_pad, *queue_video_pad, *queue_app_pad;
  GstAudioInfo info;
  GstCaps *audio_caps;
  GstBus *bus;

  /* Initialize custom data structure */
  memset (&data, 0, sizeof (data));
  data.b = 1; /* For waveform generation */
  data.d = 1;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Create the elements */
  data.app_source = gst_element_factory_make ("appsrc", "audio_source");
  data.tee = gst_element_factory_make ("tee", "tee");
  data.audio_queue = gst_element_factory_make ("queue", "audio_queue");
  data.audio_convert1 = gst_element_factory_make ("audioconvert", "audio_convert1");
  data.audio_resample = gst_element_factory_make ("audioresample", "audio_resample");
  data.audio_sink = gst_element_factory_make ("autoaudiosink", "audio_sink");
  data.video_queue = gst_element_factory_make ("queue", "video_queue");
  data.audio_convert2 = gst_element_factory_make ("audioconvert", "audio_convert2");
  data.visual = gst_element_factory_make ("wavescope", "visual");
  data.video_convert = gst_element_factory_make ("videoconvert", "video_convert");
  data.video_sink = gst_element_factory_make ("autovideosink", "video_sink");
  data.app_queue = gst_element_factory_make ("queue", "app_queue");
  data.app_sink = gst_element_factory_make ("appsink", "app_sink");

  /* Create the empty pipeline */
  data.pipeline = gst_pipeline_new ("test-pipeline");

  if (!data.pipeline || !data.app_source || !data.tee || !data.audio_queue || !data.audio_convert1 ||
      !data.audio_resample || !data.audio_sink || !data.video_queue || !data.audio_convert2 || !data.visual ||
      !data.video_convert || !data.video_sink || !data.app_queue || !data.app_sink) {
    g_printerr ("Not all elements could be created.\n");
    return -1;
  }

  /* Configure wavescope */
  g_object_set (data.visual, "shader", 0, "style", 0, NULL);

  /* Configure appsrc */
  gst_audio_info_set_format (&info, GST_AUDIO_FORMAT_S16, SAMPLE_RATE, 1, NULL);
  audio_caps = gst_audio_info_to_caps (&info);
  g_object_set (data.app_source, "caps", audio_caps, "format", GST_FORMAT_TIME, NULL);
  g_signal_connect (data.app_source, "need-data", G_CALLBACK (start_feed), &data);
  g_signal_connect (data.app_source, "enough-data", G_CALLBACK (stop_feed), &data);

  /* Configure appsink */
  g_object_set (data.app_sink, "emit-signals", TRUE, "caps", audio_caps, NULL);
  g_signal_connect (data.app_sink, "new-sample", G_CALLBACK (new_sample), &data);
  gst_caps_unref (audio_caps);

  /* Link all elements that can be automatically linked because they have "Always" pads */
  gst_bin_add_many (GST_BIN (data.pipeline), data.app_source, data.tee, data.audio_queue, data.audio_convert1, data.audio_resample,
      data.audio_sink, data.video_queue, data.audio_convert2, data.visual, data.video_convert, data.video_sink, data.app_queue,
      data.app_sink, NULL);
  if (gst_element_link_many (data.app_source, data.tee, NULL) != TRUE ||
      gst_element_link_many (data.audio_queue, data.audio_convert1, data.audio_resample, data.audio_sink, NULL) != TRUE ||
      gst_element_link_many (data.video_queue, data.audio_convert2, data.visual, data.video_convert, data.video_sink, NULL) != TRUE ||
      gst_element_link_many (data.app_queue, data.app_sink, NULL) != TRUE) {
    g_printerr ("Elements could not be linked.\n");
    gst_object_unref (data.pipeline);
    return -1;
  }

  /* Manually link the Tee, which has "Request" pads */
  tee_audio_pad = gst_element_request_pad_simple (data.tee, "src_%u");
  g_print ("Obtained request pad %s for audio branch.\n", gst_pad_get_name (tee_audio_pad));
  queue_audio_pad = gst_element_get_static_pad (data.audio_queue, "sink");
  tee_video_pad = gst_element_request_pad_simple (data.tee, "src_%u");
  g_print ("Obtained request pad %s for video branch.\n", gst_pad_get_name (tee_video_pad));
  queue_video_pad = gst_element_get_static_pad (data.video_queue, "sink");
  tee_app_pad = gst_element_request_pad_simple (data.tee, "src_%u");
  g_print ("Obtained request pad %s for app branch.\n", gst_pad_get_name (tee_app_pad));
  queue_app_pad = gst_element_get_static_pad (data.app_queue, "sink");
  if (gst_pad_link (tee_audio_pad, queue_audio_pad) != GST_PAD_LINK_OK ||
      gst_pad_link (tee_video_pad, queue_video_pad) != GST_PAD_LINK_OK ||
      gst_pad_link (tee_app_pad, queue_app_pad) != GST_PAD_LINK_OK) {
    g_printerr ("Tee could not be linked\n");
    gst_object_unref (data.pipeline);
    return -1;
  }
  gst_object_unref (queue_audio_pad);
  gst_object_unref (queue_video_pad);
  gst_object_unref (queue_app_pad);

  /* Instruct the bus to emit signals for each received message, and connect to the interesting signals */
  bus = gst_element_get_bus (data.pipeline);
  gst_bus_add_signal_watch (bus);
  g_signal_connect (G_OBJECT (bus), "message::error", (GCallback)error_cb, &data);
  gst_object_unref (bus);

  /* Start playing the pipeline */
  gst_element_set_state (data.pipeline, GST_STATE_PLAYING);

  /* Create a GLib Main Loop and set it to run */
  data.main_loop = g_main_loop_new (NULL, FALSE);
  g_main_loop_run (data.main_loop);

  /* Release the request pads from the Tee, and unref them */
  gst_element_release_request_pad (data.tee, tee_audio_pad);
  gst_element_release_request_pad (data.tee, tee_video_pad);
  gst_element_release_request_pad (data.tee, tee_app_pad);
  gst_object_unref (tee_audio_pad);
  gst_object_unref (tee_video_pad);
  gst_object_unref (tee_app_pad);

  /* Free resources */
  gst_element_set_state (data.pipeline, GST_STATE_NULL);
  gst_object_unref (data.pipeline);
  return 0;
}
```

## 加入appsrc和appsink 
首先要先設定appsrc的caps，他決定了appsrc所輸出的資料類型。我們可以用字串來建立`GstCaps`物件，只需要用`gst_caps_from_string()`函式。

另外我們還必須連接`need-data`和`enough-data`訊號。這兩個訊號是`appsrc`發出來的。
```c
/* Configure appsrc */
gst_audio_info_set_format (&info, GST_AUDIO_FORMAT_S16, SAMPLE_RATE, 1, NULL);
audio_caps = gst_audio_info_to_caps (&info);
g_object_set (data.app_source, "caps", audio_caps, NULL);
g_signal_connect (data.app_source, "need-data", G_CALLBACK (start_feed), &data);
g_signal_connect (data.app_source, "enough-data", G_CALLBACK (stop_feed), &data);
```

## new-sample訊號
另外我們還要接上`app_sink`的訊號`new-sample`，這個訊號預設是關閉的所以必須手動打開這個訊號，由`emit-signals`可以設定。
```c
/* Configure appsink */
g_object_set (data.app_sink, "emit-signals", TRUE, "caps", audio_caps, NULL);
g_signal_connect (data.app_sink, "new-sample", G_CALLBACK (new_sample), &data);
gst_caps_unref (audio_caps);
```

## callback function
我們的callback function在每當`appsrc`內部的queue快要沒資料的時候被呼叫。他唯一做的事情就是註冊一個GLib的函式`g_idle_add()`，他將會給`appsrc`資料直到`appsrc`滿了為止。  

Glib的main event loop更進一步說明可以參考[這裡](https://zhuanlan.zhihu.com/p/567441726)  

我們將`g_idle_add()`回傳的id做個紀錄以便等一下可以停止他。
```c
/* This signal callback triggers when appsrc needs data. Here, we add an idle handler
 * to the mainloop to start pushing data into the appsrc */
static void start_feed (GstElement *source, guint size, CustomData *data) {
  if (data->sourceid == 0) {
    g_print ("Start feeding\n");
    data->sourceid = g_idle_add ((GSourceFunc) push_data, data);
  }
}
```

下面這個callback function當`appsrc`內部的queue滿的時候會被呼叫。在這裡我們就直接用`g_source_remove()`移除idle function
```c
/* This callback triggers when appsrc has enough data and we can stop sending.
 * We remove the idle handler from the mainloop */
static void stop_feed (GstElement *source, CustomData *data) {
  if (data->sourceid != 0) {
    g_print ("Stop feeding\n");
    g_source_remove (data->sourceid);
    data->sourceid = 0;
  }
}
```

接下來這個function是推送資料給`appsrc`的callback，是給GLib的`g_idle_add()`呼叫用的。

首先他的工作是建立一個新的buffer並且給定大小(這個範例設為1024 bytes)，設定大小可以用`gst_buffer_new_and_alloc()`。

接下例我們計算我們已經餵給`appsrc`的資料數目，並且用`CustomData.num_samples`紀錄，如此一來就可以給這個buffer時間戳，時皆戳可以用`GstBuffer`的`GST_BUFFER_TIMESTAMP` marco。

因為我們每次都提供相同大小的buffer，他的長度都是相同的，我們可以用`GstBuffer`的`GST_BUFFER_DURATION`marco來設定duration。

`gst_util_uint64_scale()`是用來放大或縮小大數字的函式，用這個函是就不用擔心overflows。

buffer的大小可以用GstBuffer 可以用GST_BUFFER_DATA 取得。

```c
/* This method is called by the idle GSource in the mainloop, to feed CHUNK_SIZE bytes into appsrc.
 * The ide handler is added to the mainloop when appsrc requests us to start sending data (need-data signal)
 * and is removed when appsrc has enough data (enough-data signal).
 */
static gboolean push_data (CustomData *data) {
  GstBuffer *buffer;
  GstFlowReturn ret;
  int i;
  gint16 *raw;
  gint num_samples = CHUNK_SIZE / 2; /* Because each sample is 16 bits */
  gfloat freq;

  /* Create a new empty buffer */
  buffer = gst_buffer_new_and_alloc (CHUNK_SIZE);

  /* Set its timestamp and duration */
  GST_BUFFER_TIMESTAMP (buffer) = gst_util_uint64_scale (data->num_samples, GST_SECOND, SAMPLE_RATE);
  GST_BUFFER_DURATION (buffer) = gst_util_uint64_scale (num_samples, GST_SECOND, SAMPLE_RATE);

  /* Generate some psychodelic waveforms */
  raw = (gint16 *)GST_BUFFER_DATA (buffer);
```

最後就是把生成好的資料推送進`appsrc`裡面，並且會觸發`push-buffer`訊號。
```c
/* Push the buffer into the appsrc */
g_signal_emit_by_name (data->app_source, "push-buffer", buffer, &ret);

/* Free the buffer now that we are done with it */
gst_buffer_unref (buffer);
```

而當`appsink`接收到資料的時候下面的函式會被呼叫。我們用`pull-sample`動作訊號來取得buffer並且應到螢幕上。利用`GST_BUFFER_DATA`取得資料的指針以及`GST_BUFFER_SIZE`取得資料大小。

注意，在這裡的buffer不一定會跟我們在前面指定的buffer大小一樣，因為任何element都有可能去更動buffer，雖然在這個範例buffer並沒有被改動。

# Debugging tools
## debug log
debug log是由`GST_DEBUG`環境變數控制。下面是GST_DEBUG=2的時候的debug log。
```
0:00:00.868050000  1592   09F62420 WARN                 filesrc gstfilesrc.c:1044:gst_file_src_start:<filesrc0> error: No such file "non-existing-file.webm"
```
通常我們不會把debug log全部打開以免訊息塞爆文件或是訊息視窗。下面是各種等級的debug log會輸出的資料。
```
| # | Name    | Description                                                    |
|---|---------|----------------------------------------------------------------|
| 0 | none    | No debug information is output.                                |
| 1 | ERROR   | Logs all fatal errors. These are errors that do not allow the  |
|   |         | core or elements to perform the requested action. The          |
|   |         | application can still recover if programmed to handle the      |
|   |         | conditions that triggered the error.                           |
| 2 | WARNING | Logs all warnings. Typically these are non-fatal, but          |
|   |         | user-visible problems are expected to happen.                  |
| 3 | FIXME   | Logs all "fixme" messages. Those typically that a codepath that|
|   |         | is known to be incomplete has been triggered. It may work in   |
|   |         | most cases, but may cause problems in specific instances.      |
| 4 | INFO    | Logs all informational messages. These are typically used for  |
|   |         | events in the system that only happen once, or are important   |
|   |         | and rare enough to be logged at this level.                    |
| 5 | DEBUG   | Logs all debug messages. These are general debug messages for  |
|   |         | events that happen only a limited number of times during an    |
|   |         | object's lifetime; these include setup, teardown, change of    |
|   |         | parameters, etc.                                               |
| 6 | LOG     | Logs all log messages. These are messages for events that      |
|   |         | happen repeatedly during an object's lifetime; these include   |
|   |         | streaming and steady-state conditions. This is used for log    |
|   |         | messages that happen on every buffer in an element for example.|
| 7 | TRACE   | Logs all trace messages. Those are message that happen very    |
|   |         | very often. This is for example is each time the reference     |
|   |         | count of a GstMiniObject, such as a GstBuffer or GstEvent, is  |
|   |         | modified.                                                      |
| 9 | MEMDUMP | Logs all memory dump messages. This is the heaviest logging and|
|   |         | may include dumping the content of blocks of memory.           |
+------------------------------------------------------------------------------+
```

## 設置element的debug log
如果要設置個別element的debug level，範例如下。如此一來`audiotestsrc`的level是6，其他的都是2
```
GST_DEBUG=2,audiotestsrc:6
```

## GST_DEBUG格是
GST_DEBUG的第一個參數是optional的，他會設定全域的debug level。參數之間以逗號`,`區隔，除了第一個參數，每一個設定的格式都是`category:level`。

`*`也可以被用在GST_DEBUG裡面，例如`GST_DEBUG=2,audio*:6`會把所有開頭為audio的element都設為level 6，其他保持level 2。

## 增加自訂除錯訊息
如果要讓category 看起來更有意義，可以加入下面兩行
```c
GST_DEBUG_CATEGORY_STATIC (my_category);
#define GST_CAT_DEFAULT my_category
```
以及下面這行在`gst_init()`被呼叫之後。
```c
GST_DEBUG_CATEGORY_INIT (my_category, "my category", 0, "This is my very own");
```

## 取得pipeline的圖
gstreamer可以將你的pipeline輸出成.dot檔可以用例如GraphViz來開啟。

如果在程式內想開啟這項功能可以用`GST_DEBUG_BIN_TO_DOT_FILE()` 和 `GST_DEBUG_BIN_TO_DOT_FILE_WITH_TS()`marco來開啟這個功能。
範例:  
可以在程式中加入下面程式碼來儲存pipeline圖  
[官方文件](https://gstreamer.freedesktop.org/documentation/gstreamer/debugutils.html#gst_debug_bin_to_dot_file)  
[圖案參數](https://gstreamer.freedesktop.org/documentation/gstreamer/debugutils.html?gi-language=c#GstDebugGraphDetails)  
[參考來源](https://github.com/nnstreamer/nnstreamer/blob/main/tools/debugging/README.md)
```c
GST_DEBUG_BIN_TO_DOT_FILE(pipeline, GST_DEBUG_GRAPH_SHOW_ALL, "pipeline"); //三個參數分別為 pipeline的instance, 圖案參數, 檔案名稱
```
這行程式記得加在將pipeline state設定為`playing`之前，這樣才能完整看到pipeline的樣子。

然後執行程式之前先設定環境變數`GST_DEBUG_DUMP_DOT_DIR`來指定儲存位置
```bash
export GST_DEBUG_DUMP_DOT_DIR=./tracing/
mkdir tracing
```

接下來就會在tracing看到`pipeline.dot`檔

安裝`xdot`來讀取.dot檔
```bash
sudo apt install xdot
xdot pipeline.dot
```


gst-launch-1.0可以用用GST_DEBUG_DUMP_DOT_DIR來設定儲存圖片的資料夾並開啟這項功能。每當pipeline的狀態改變的時候都會畫一張圖，如此一來就可以看到pipeline的變化


# gst-debugger
[遠端除錯gstreamer](https://github.com/GNOME/gst-debugger.git)

安裝protoc  
http://google.github.io/proto-lens/installing-protoc.html

# Streaming
這裡將介紹streaming要注意的點
* 開啟buffering
* 斷線重連
通常網路串流會因為網路連線的關係造成串流封包沒有準時到達而造成跨面卡住。而這個問題的解法就是使用`buffer`。`buffer`讓一些影音chunks儲存在queue裡面，如此一來雖然剛開始影片會稍微延遲一點，但是如果網路連線不穩的話話面不會卡住，因為queue裡面還有chunks。

## clock
應用程式應該隨時監看`buffer`的狀態，如果`buffer`太少，就應該暫停撥放。為了達到所有的sink都可以同步，GStreamer有一個global clock，所有的element都會共用這個global clock。
有時候如果切換streaming或是切換輸出裝置，clock會消失，這時候就必須重選clock，下面的範例將會解說這個步驟。

當clock消失的時候應用程式會接收到訊息，這時候只需要將pipeline設為PAUSED再設為PLAYING就可以選擇新的clock。

## 範例
範例`basic-tutorial-12.c`
```c
#include <gst/gst.h>
#include <string.h>

typedef struct _CustomData {
  gboolean is_live;
  GstElement *pipeline;
  GMainLoop *loop;
} CustomData;

static void cb_message (GstBus *bus, GstMessage *msg, CustomData *data) {

  switch (GST_MESSAGE_TYPE (msg)) {
    case GST_MESSAGE_ERROR: {
      GError *err;
      gchar *debug;

      gst_message_parse_error (msg, &err, &debug);
      g_print ("Error: %s\n", err->message);
      g_error_free (err);
      g_free (debug);

      gst_element_set_state (data->pipeline, GST_STATE_READY);
      g_main_loop_quit (data->loop);
      break;
    }
    case GST_MESSAGE_EOS:
      /* end-of-stream */
      gst_element_set_state (data->pipeline, GST_STATE_READY);
      g_main_loop_quit (data->loop);
      break;
    case GST_MESSAGE_BUFFERING: {
      gint percent = 0;

      /* If the stream is live, we do not care about buffering. */
      if (data->is_live) break;

      gst_message_parse_buffering (msg, &percent);
      g_print ("Buffering (%3d%%)\r", percent);
      /* Wait until buffering is complete before start/resume playing */
      if (percent < 100)
        gst_element_set_state (data->pipeline, GST_STATE_PAUSED);
      else
        gst_element_set_state (data->pipeline, GST_STATE_PLAYING);
      break;
    }
    case GST_MESSAGE_CLOCK_LOST:
      /* Get a new clock */
      gst_element_set_state (data->pipeline, GST_STATE_PAUSED);
      gst_element_set_state (data->pipeline, GST_STATE_PLAYING);
      break;
    default:
      /* Unhandled message */
      break;
    }
}

int main(int argc, char *argv[]) {
  GstElement *pipeline;
  GstBus *bus;
  GstStateChangeReturn ret;
  GMainLoop *main_loop;
  CustomData data;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Initialize our data structure */
  memset (&data, 0, sizeof (data));

  /* Build the pipeline */
  pipeline = gst_parse_launch ("playbin uri=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm", NULL);
  bus = gst_element_get_bus (pipeline);

  /* Start playing */
  ret = gst_element_set_state (pipeline, GST_STATE_PLAYING);
  if (ret == GST_STATE_CHANGE_FAILURE) {
    g_printerr ("Unable to set the pipeline to the playing state.\n");
    gst_object_unref (pipeline);
    return -1;
  } else if (ret == GST_STATE_CHANGE_NO_PREROLL) {
    data.is_live = TRUE;
  }

  main_loop = g_main_loop_new (NULL, FALSE);
  data.loop = main_loop;
  data.pipeline = pipeline;

  gst_bus_add_signal_watch (bus);
  g_signal_connect (bus, "message", G_CALLBACK (cb_message), &data);

  g_main_loop_run (main_loop);

  /* Free resources */
  g_main_loop_unref (main_loop);
  gst_object_unref (bus);
  gst_element_set_state (pipeline, GST_STATE_NULL);
  gst_object_unref (pipeline);
  return 0;
}
```

## 說明
在這個範例中比較特別的是下面這一段，注意如果收到`GST_STATE_CHANGE_NO_PREROLL`而不是`GST_STATE_CHANGE_SUCCESS`，這代表目前正再撥放直撥串流。

因為直撥串流是不能暫停的，所以就算把pipeline的狀態設為PAUSED他的行為還是跟PLAYING一樣。而且即使我們嘗試將pipeline設為PLAYING也會收到這個訊息。

因為我們想要關閉直撥串流的`buffering`，所以我們用`gst_element_set_state()`在data裡面做記號
```c
/* Start playing */
ret = gst_element_set_state (pipeline, GST_STATE_PLAYING);
if (ret == GST_STATE_CHANGE_FAILURE) {
  g_printerr ("Unable to set the pipeline to the playing state.\n");
  gst_object_unref (pipeline);
  return -1;
} else if (ret == GST_STATE_CHANGE_NO_PREROLL) {
  data.is_live = TRUE;
}
```

## callback
接下來我們看一下message parsing callback，首先如果發現是直撥串流，就不要打開`buffering`。接下來用`gst_message_parse_buffering()`來取得 buffering level。

然後我們印出 buffering level並且設定當 buffering level小於100%的時候就暫停pipeline，如果超過就設為PLAYING。

程式執行的時候會看到buffer慢慢攀升到100%，然後如果網路不穩到buffer小於100%，就會暫停撥放直到回復100%後才重新撥放。
```c
case GST_MESSAGE_BUFFERING: {
  gint percent = 0;

  /* If the stream is live, we do not care about buffering. */
  if (data->is_live) break;

  gst_message_parse_buffering (msg, &percent);
  g_print ("Buffering (%3d%%)\r", percent);
  /* Wait until buffering is complete before start/resume playing */
  if (percent < 100)
    gst_element_set_state (data->pipeline, GST_STATE_PAUSED);
  else
    gst_element_set_state (data->pipeline, GST_STATE_PLAYING);
  break;
}
```


## lost clock
另一個我們要處理的消息是遺失clock，我們只需要將pipeline設為PAUSED再設為PLAYING就可以了。
```c
case GST_MESSAGE_CLOCK_LOST:
  /* Get a new clock */
  gst_element_set_state (data->pipeline, GST_STATE_PAUSED);
  gst_element_set_state (data->pipeline, GST_STATE_PLAYING);
  break;
```