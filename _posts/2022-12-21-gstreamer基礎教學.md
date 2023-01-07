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
element是GStreamer最基礎的元素，影音資料從`source`流向`sink`，過程中經過`filter`對影音資料進行處理，這三個元素組成為`pipeline`  

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
Gstreamer的[bin](https://gstreamer.freedesktop.org/documentation/gstreamer/gstbin.html?gi-language=c#GstBin)是用來裝入element的。

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

## 
