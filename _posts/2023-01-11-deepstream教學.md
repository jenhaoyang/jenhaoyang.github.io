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


## Deepstream 說明書
https://docs.nvidia.com/metropolis/deepstream/dev-guide/text/DS_plugin_gst-nvdsxfer.html

## gst-launch-1.0建立rtsp輸入源的pipeline
首先先用`gst-launch-1.0`建立一個簡單的rtsp輸入、螢幕輸出的pipeline

```
gst-launch-1.0 rtspsrc location='rtsp://192.168.1.10:554/user=admin_password=xxxxxx_channel=1_stream=0.sdp' ! rtph264depay ! h264parse ! nvv4l2decoder ! nvvideoconvert ! video/x-raw,format=BGRx ! videoconvert ! video/x-raw,format=BGR ! autovideosink
```

## 將python的範例程式轉成c++
https://github.com/NVIDIA-AI-IOT/deepstream_python_apps/tree/master/apps/deepstream-rtsp-in-rtsp-out


## 建立mjpeg串流
1. 指令方式
```
gst-launch-1.0 -v rtspsrc location="rtsp://<rtsp url>/live1.sdp" \
! rtph264depay ! avdec_h264 \
! timeoverlay halignment=right valignment=bottom \
! videorate ! video/x-raw,framerate=37000/1001 ! jpegenc ! multifilesink location="snapshot.jpeg"
```
https://stackoverflow.com/questions/59885450/jpeg-live-stream-in-html-slow


## 查詢deepstream bin的說明
```
gst-inspect-1.0 nvurisrcbin
```

## gst-launch-1.0輸出除錯訊息到檔案
參考:  
https://www.cnblogs.com/xleng/p/12228720.html
```
GST_DEBUG_NO_COLOR=1 GST_DEBUG_FILE=pipeline.log GST_DEBUG=5 gst-launch-1.0 -v rtspsrc location="rtsp://192.168.8.19/live.sdp" user-id="root" user-pw="3edc\$RFV" ! rtph264depay ! avdec_h264 ! timeoverlay halignment=right valignment=bottom ! videorate ! video/x-raw,framerate=37000/1001 ! jpegenc ! multifilesink location="snapshot.jpeg"
```

參考:
https://gstreamer.freedesktop.org/documentation/tutorials/basic/debugging-tools.html?gi-language=c  
https://embeddedartistry.com/blog/2018/02/22/generating-gstreamer-pipeline-graphs/  

## 本地端觀看udp傳送影像
host設為本機ip或127.0.0.1

send:
```
gst-launch-1.0 -v videotestsrc ! x264enc tune=zerolatency bitrate=500 speed-preset=superfast ! rtph264pay ! udpsink port=5000 host=$HOST
```

receive:
```
gst-launch-1.0 -v udpsrc port=5000 ! "application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H264, payload=(int)96" ! rtph264depay ! h264parse ! avdec_h264 ! videoconvert ! autovideosink
```

## Glibs說明書
http://irtfweb.ifa.hawaii.edu/SoftwareDocs/gtk20/glib/glib-hash-tables.html#g-int-hash

## GDB文字圖形介面
https://blog.louie.lu/2016/09/12/gdb-%E9%8C%A6%E5%9B%8A%E5%A6%99%E8%A8%88/

## 範例
https://gist.github.com/liviaerxin/bb34725037fd04afa76ef9252c2ee875#tips-for-debug

## rtsp 元件nvrtspoutsinkbin
nvrtspoutsinkbin沒有說明書，只能用gst-inspect-1.0看
https://forums.developer.nvidia.com/t/where-can-fine-nvrtspoutsinkbin-info/199124  

範例
/opt/nvidia/deepstream/deepstream/sources/apps/sample_apps/deepstream_reference_apps/deepstream-bodypose-3d/sources/deepstream_pose_estimation_app.cpp

```c
    /* Create RTSP output bin */
    rtsp_out_bin = gst_element_factory_make ("nvrtspoutsinkbin", "nvrtsp-renderer");

    if (!rtsp_out_bin) {
      g_printerr ("Failed to create RTSP output elements. Exiting.\n");
      return -1;
    }

    g_object_set (G_OBJECT (rtsp_out_bin), "sync", TRUE, NULL);
    g_object_set (G_OBJECT (rtsp_out_bin), "bitrate", 768000, NULL);
    g_object_set (G_OBJECT (rtsp_out_bin), "rtsp-port", rtsp_port_num, NULL);
    g_object_set (G_OBJECT (rtsp_out_bin), "enc-type", enc_type, NULL);

    gst_bin_add_many (GST_BIN (pipeline), rtsp_out_bin, NULL);
```

## 取得source id
[https://forums.developer.nvidia.com/t/how-to-get-sources-index-in-deepstream/244461  ](https://forums.developer.nvidia.com/t/can-i-pass-source-id-from-nvinferserver-along-with-frame/205418)

可以用prob取得meta data
deepstream_test3_app.c 有範例

[probe使用範例 ](https://forums.developer.nvidia.com/t/how-to-make-metadata-probe-for-classification-only-pipeline/171126) 

![metadata](/assets/img/deepstream/metadata.png)



## 切換輸入源
https://forums.developer.nvidia.com/t/how-switch-camera-output-gst-nvmultistreamtiler/233062

```
tiler_sink_pad.add_probe(Gst.PadProbeType.BUFFER, tiler_sink_pad_buffer_probe, 0)

tiler.set_property("show-source", <stream_id>) `
```

/opt/nvidia/deepstream/deepstream/sources/apps/apps-common/src/deepstream-yaml/deepstream_source_yaml.cpp有範例


## 斷線重連
rust的插件(可能可以編譯成c函式庫)
https://coaxion.net/blog/2020/07/automatic-retry-on-error-and-fallback-stream-handling-for-gstreamer-sources/  

https://gitlab.freedesktop.org/gstreamer/gst-plugins-rs/-/tree/master/utils/fallbackswitch

編譯rust插件  
https://www.collabora.com/news-and-blog/blog/2020/06/23/cross-building-rust-gstreamer-plugins-for-the-raspberry-pi/  

RUST說明書
https://rust-lang.tw/book-tw/ch01-03-hello-cargo.html  



## 截出有物件的圖
https://forums.developer.nvidia.com/t/saving-frame-with-detected-object-jetson-nano-ds4-0-2/121797/3

## 關閉Ubuntu圖形介面
https://linuxconfig.org/how-to-disable-enable-gui-on-boot-in-ubuntu-20-04-focal-fossa-linux-desktop


## 關閉使用gpu的資源
https://heary.cn/posts/Linux环境下重装NVIDIA驱动报错kernel-module-nvidia-modeset-in-use问题分析/

發現nvidia smi persistence mode會占用GPU資源，必須釋放掉才能安裝新的driver
可以用nvidia-smi的指令關掉https://docs.nvidia.com/deploy/driver-persistence/index.html#usage
```
nvidia-smi -pm 0
```

## 移除舊的driver
```
apt-get remove --purge nvidia-driver-520
apt-get autoremove
```


## queue的用途
https://docs.xilinx.com/r/en-US/ug1449-multimedia/Performance-Improvement-from-the-GStreamer-Perspective

## probe
https://coaxion.net/blog/2014/01/gstreamer-dynamic-pipelines/  

https://erit-lvx.medium.com/probes-handling-in-gstreamer-pipelines-3f96ea367f31


## deepstream-test4 用prob取得metadata的範例
NvDsBatchMeta資料圖:
https://docs.nvidia.com/metropolis/deepstream/dev-guide/text/DS_plugin_metadata.html
```c
static GstPadProbeReturn
osd_sink_pad_buffer_probe (GstPad * pad, GstPadProbeInfo * info,
    gpointer u_data)
{
  GstBuffer *buf = (GstBuffer *) info->data;
  NvDsFrameMeta *frame_meta = NULL;
  NvOSD_TextParams *txt_params = NULL;
  guint vehicle_count = 0;
  guint person_count = 0;
  gboolean is_first_object = TRUE;
  NvDsMetaList *l_frame, *l_obj;

  NvDsBatchMeta *batch_meta = gst_buffer_get_nvds_batch_meta (buf);
  if (!batch_meta) {
    // No batch meta attached.
    return GST_PAD_PROBE_OK;
  }

  //batch_meta : NvDsBatchMeta https://docs.nvidia.com/metropolis/deepstream/sdk-api/struct__NvDsBatchMeta.html
  //
  //l_frame : NvDsFrameMetaList, 本質是GList http://irtfweb.ifa.hawaii.edu/SoftwareDocs/gtk20/glib/glib-doubly-linked-lists.html#GList
  //   struct GList
  // {
  //   gpointer data;
  //   GList *next;
  //   GList *prev;
  // };

  for (l_frame = batch_meta->frame_meta_list; l_frame; l_frame = l_frame->next) {
    frame_meta = (NvDsFrameMeta *) l_frame->data;

    if (frame_meta == NULL) {
      // Ignore Null frame meta.
      continue;
    }

    is_first_object = TRUE;

    // frame_meta : NvDsFrameMeta https://docs.nvidia.com/metropolis/deepstream/sdk-api/struct__NvDsFrameMeta.html
    // l_obj : NvDsObjectMetaList * 本質是GList 
    // obj_meta : NvDsObjectMeta
    for (l_obj = frame_meta->obj_meta_list; l_obj; l_obj = l_obj->next) {
      NvDsObjectMeta *obj_meta = (NvDsObjectMeta *) l_obj->data;

      if (obj_meta == NULL) {
        // Ignore Null object.
        continue;
      }

      // obj_meta : NvDsObjectMeta
      // text_params : NvOSD_TextParams 描述物件的文字
      // line233 - 241應該是清掉原本的文字然後放入字定義的class名稱
      txt_params = &(obj_meta->text_params);
      if (txt_params->display_text)
        g_free (txt_params->display_text);

      txt_params->display_text = g_malloc0 (MAX_DISPLAY_LEN);

      g_snprintf (txt_params->display_text, MAX_DISPLAY_LEN, "%s ",
          pgie_classes_str[obj_meta->class_id]);

      if (obj_meta->class_id == PGIE_CLASS_ID_VEHICLE)
        vehicle_count++;
      if (obj_meta->class_id == PGIE_CLASS_ID_PERSON)
        person_count++;

      /* Now set the offsets where the string should appear */
      txt_params->x_offset = obj_meta->rect_params.left;
      txt_params->y_offset = obj_meta->rect_params.top - 25;

      /* Font , font-color and font-size */
      txt_params->font_params.font_name = "Serif";
      txt_params->font_params.font_size = 10;
      txt_params->font_params.font_color.red = 1.0;
      txt_params->font_params.font_color.green = 1.0;
      txt_params->font_params.font_color.blue = 1.0;
      txt_params->font_params.font_color.alpha = 1.0;

      /* Text background color */
      txt_params->set_bg_clr = 1;
      txt_params->text_bg_clr.red = 0.0;
      txt_params->text_bg_clr.green = 0.0;
      txt_params->text_bg_clr.blue = 0.0;
      txt_params->text_bg_clr.alpha = 1.0;

      /*
       * Ideally NVDS_EVENT_MSG_META should be attached to buffer by the
       * component implementing detection / recognition logic.
       * Here it demonstrates how to use / attach that meta data.
       */
      if (is_first_object && !(frame_number % frame_interval)) {
        /* Frequency of messages to be send will be based on use case.
         * Here message is being sent for first object every frame_interval(default=30).
         */

        NvDsEventMsgMeta *msg_meta =
            (NvDsEventMsgMeta *) g_malloc0 (sizeof (NvDsEventMsgMeta));
        msg_meta->bbox.top = obj_meta->rect_params.top;
        msg_meta->bbox.left = obj_meta->rect_params.left;
        msg_meta->bbox.width = obj_meta->rect_params.width;
        msg_meta->bbox.height = obj_meta->rect_params.height;
        msg_meta->frameId = frame_number;
        msg_meta->trackingId = obj_meta->object_id;
        msg_meta->confidence = obj_meta->confidence;
        generate_event_msg_meta (msg_meta, obj_meta->class_id, obj_meta);

        // 要增加自訂的meta data必須要先用            nvds_acquire_user_meta_from_pool (batch_meta);取得
        // https://docs.nvidia.com/metropolis/deepstream/dev-guide/text/DS_plugin_metadata.html#user-custom-metadata-addition-inside-nvdsbatchmeta

        NvDsUserMeta *user_event_meta =
            nvds_acquire_user_meta_from_pool (batch_meta);
        if (user_event_meta) {
          user_event_meta->user_meta_data = (void *) msg_meta;
          user_event_meta->base_meta.meta_type = NVDS_EVENT_MSG_META;
          user_event_meta->base_meta.copy_func =
              (NvDsMetaCopyFunc) meta_copy_func;
          user_event_meta->base_meta.release_func =
              (NvDsMetaReleaseFunc) meta_free_func;
          nvds_add_user_meta_to_frame (frame_meta, user_event_meta);
        } else {
          g_print ("Error in attaching event meta to buffer\n");
        }
        is_first_object = FALSE;
      }
    }
  }
  g_print ("Frame Number = %d "
      "Vehicle Count = %d Person Count = %d\n",
      frame_number, vehicle_count, person_count);
  frame_number++;

  return GST_PAD_PROBE_OK;
}

```
## 注意element的名稱不要一樣以免出錯

參考:  
https://www.gclue.jp/2022/06/gstreamer.html  