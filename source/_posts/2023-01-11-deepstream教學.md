---
layout: post
title: deepstream教學
date: 2023-01-11 12:28 +0800
categories: [深度學習工具]
tags: [deepstream]     # TAG names should always be lowercase
---


# 客製化模型實作nvinfer介面

## nvinfer呼叫介面
任何客製化介面最終必須被編譯成一個獨立的shared library。nvinfer在執行期間利用`dlopen()`呼叫函式庫，並且利用`dlsym()`呼叫函式庫中的函式。進一步的資訊紀錄在`nvdsinfer_custom_impl.h`裡面https://docs.nvidia.com/metropolis/deepstream/sdk-api/nvdsinfer__custom__impl_8h.html  






## 客製化Output Parsing
* 對於detectors使用者必須自行解析模型的輸出並且將之轉化成bounding box 座標和物件類別。對於classifiers則是必須自行解析出物件屬性。範例在`/opt/nvidia/deepstream/deepstream/sources/libs/nvdsinfer_customparser`，裡面的README有關於使用custom parser的說明。  
* 客製化parsing function必須為`NvDsInferParseCustomFunc`型態。在`nvdsinfer_custom_impl.h`的221行可以看到下面的型態定義，代表每一個客製化的解析函式都必須符合這個格式
```c
/**
 * Type definition for the custom bounding box parsing function.
 *
 * @param[in]  outputLayersInfo A vector containing information on the output
 *                              layers of the model.
 * @param[in]  networkInfo      Network information.
 * @param[in]  detectionParams  Detection parameters required for parsing
 *                              objects.
 * @param[out] objectList       A reference to a vector in which the function
 *                              is to add parsed objects.
 */
typedef bool (* NvDsInferParseCustomFunc) (
        std::vector<NvDsInferLayerInfo> const &outputLayersInfo,
        NvDsInferNetworkInfo  const &networkInfo,
        NvDsInferParseDetectionParams const &detectionParams,
        std::vector<NvDsInferObjectDetectionInfo> &objectList);
```


* 客製化parsing function可以在`Gst-nvinfer`的參數檔`parse-bbox-func-name`和`custom-lib-name`屬性指定。例如我們定義了Yolov2-tiny的客製化bounding box解析函式`NvDsInferParseCustomYoloV2Tiny`，編譯出來的shared library位於`nvdsinfer_custom_impl_Yolo/libnvdsinfer_custom_impl_Yolo.so`，我們在設定檔就就必須要有以下設定
```
parse-bbox-func-name=NvDsInferParseCustomYoloV2Tiny
custom-lib-path=nvdsinfer_custom_impl_Yolo/libnvdsinfer_custom_impl_Yolo.so
```

可以藉由在定義函式後呼叫`CHECK_CUSTOM_PARSE_FUNC_PROTOTYPE()`marco來驗證函式的定義。
使用範例如下
```c
extern "C" bool NvDsInferParseCustomYoloV2Tiny(
    std::vector<NvDsInferLayerInfo> const& outputLayersInfo,
    NvDsInferNetworkInfo const& networkInfo,
    NvDsInferParseDetectionParams const& detectionParams,
    std::vector<NvDsInferParseObjectInfo>& objectList)
{
    ...
}
CHECK_CUSTOM_PARSE_FUNC_PROTOTYPE(NvDsInferParseCustomYoloV2Tiny);
```
https://forums.developer.nvidia.com/t/deepstreamsdk-4-0-1-custom-yolov3-tiny-error/108391?u=jenhao






## IPlugin Implementation
對於TensorRT不支援的network layer，Deepstream提供IPlugin interface來客製化處理。在`/opt/nvidia/deepstream/deepstream/sources`底下的`objectDetector_SSD`, `objectDetector_FasterRCNN`, 和 `objectDetector_YoloV3`資料夾展示了如何使用custom layers。

在`objectDetector_YoloV3`範例中我們可以看到如何製作Tensorrt不支援的Yolov3的yolo layer。可以在`yolo.cpp`中看到自定義的layer是如何被呼叫使用的，程式節錄如下。
```c
....
else if (m_ConfigBlocks.at(i).at("type") == "yolo") {
            nvinfer1::Dims prevTensorDims = previous->getDimensions();
            assert(prevTensorDims.d[1] == prevTensorDims.d[2]);
            TensorInfo& curYoloTensor = m_OutputTensors.at(outputTensorCount);
            curYoloTensor.gridSize = prevTensorDims.d[1];
            curYoloTensor.stride = m_InputW / curYoloTensor.gridSize;
            m_OutputTensors.at(outputTensorCount).volume = curYoloTensor.gridSize
                * curYoloTensor.gridSize
                * (curYoloTensor.numBBoxes * (5 + curYoloTensor.numClasses));
            std::string layerName = "yolo_" + std::to_string(i);
            curYoloTensor.blobName = layerName;
            nvinfer1::IPluginV2* yoloPlugin
                = new YoloLayerV3(m_OutputTensors.at(outputTensorCount).numBBoxes,
                                  m_OutputTensors.at(outputTensorCount).numClasses,
                                  m_OutputTensors.at(outputTensorCount).gridSize);
            assert(yoloPlugin != nullptr);
            nvinfer1::IPluginV2Layer* yolo =
                network.addPluginV2(&previous, 1, *yoloPlugin);
            assert(yolo != nullptr);
            yolo->setName(layerName.c_str());
            std::string inputVol = dimsToString(previous->getDimensions());
            previous = yolo->getOutput(0);
            assert(previous != nullptr);
            previous->setName(layerName.c_str());
            std::string outputVol = dimsToString(previous->getDimensions());
            network.markOutput(*previous);
            channels = getNumChannels(previous);
            tensorOutputs.push_back(yolo->getOutput(0));
            printLayerInfo(layerIndex, "yolo", inputVol, outputVol, std::to_string(weightPtr));
            ++outputTensorCount;
        }
...
```
而其他版本的YOLO，Nvidia也已經幫我們建立好許多Plugin，例如yolov2的region layer，Nvidia已經幫我們建立，其他已經建立好的layer可以在這裡找到。
https://github.com/NVIDIA/TensorRT/tree/1c0e3fdd039c92e584430a2ed91b4e2612e375b8/plugin


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


## NvDsObjEncUsrArgs參數的功用
* bool 	isFrame : 告訴encoder要編碼整張照片還是編碼每一個偵測物件的截圖。
  * 1: Encodes the entire frame. 
  * 0: Encodes object of specified resolution.
* bool 	saveImg : 會直接儲存一張照片到當前資料夾
* bool 	attachUsrMeta : 
  * 決定是否加上NVDS_CROP_IMAGE_META metadata




# Deepstream截圖，以deepstream_image_meta_test為例
## 注意:
[根據文件](https://docs.nvidia.com/metropolis/deepstream/sdk-api/group__ee__object__encoder.html#ga83f212f9db247092c2b51b6f239f7482)`nvds_obj_enc_process`是一個非阻塞的函式，使用者必須呼叫`nvds_obj_enc_finish()`以確保所有的圖片都已經確實被處理完成。

## 第一步，設定要儲存照片的條件並且encode成jpg檔
```c
/* pgie_src_pad_buffer_probe will extract metadata received on pgie src pad
 * and update params for drawing rectangle, object information etc. We also
 * iterate through the object list and encode the cropped objects as jpeg
 * images and attach it as user meta to the respective objects.*/
static GstPadProbeReturn
pgie_src_pad_buffer_probe (GstPad * pad, GstPadProbeInfo * info, gpointer ctx)
{
  GstBuffer *buf = (GstBuffer *) info->data;
  GstMapInfo inmap = GST_MAP_INFO_INIT;
  if (!gst_buffer_map (buf, &inmap, GST_MAP_READ)) {
    GST_ERROR ("input buffer mapinfo failed");
    return GST_PAD_PROBE_DROP;
  }
  NvBufSurface *ip_surf = (NvBufSurface *) inmap.data;
  gst_buffer_unmap (buf, &inmap);

  NvDsObjectMeta *obj_meta = NULL;
  guint vehicle_count = 0;
  guint person_count = 0;
  NvDsMetaList *l_frame = NULL;
  NvDsMetaList *l_obj = NULL;
  NvDsBatchMeta *batch_meta = gst_buffer_get_nvds_batch_meta (buf);
  for (l_frame = batch_meta->frame_meta_list; l_frame != NULL;
      l_frame = l_frame->next) {
    NvDsFrameMeta *frame_meta = (NvDsFrameMeta *) (l_frame->data);
    /* For demonstration purposes, we will encode the first 10 frames. */
    if(frame_count <= 10) {
      NvDsObjEncUsrArgs frameData = { 0 };
      /* Preset */
      frameData.isFrame = 1;
      /* To be set by user */
      frameData.saveImg = save_img;
      frameData.attachUsrMeta = attach_user_meta;
      /* Set if Image scaling Required */
      frameData.scaleImg = FALSE;
      frameData.scaledWidth = 0;
      frameData.scaledHeight = 0;
      /* Quality */
      frameData.quality = 80;
      /* Main Function Call */
      nvds_obj_enc_process (ctx, &frameData, ip_surf, NULL, frame_meta);
    }
    guint num_rects = 0;
    for (l_obj = frame_meta->obj_meta_list; l_obj != NULL; l_obj = l_obj->next) {
      obj_meta = (NvDsObjectMeta *) (l_obj->data);
      if (obj_meta->class_id == PGIE_CLASS_ID_VEHICLE) {
        vehicle_count++;
        num_rects++;
      }
      if (obj_meta->class_id == PGIE_CLASS_ID_PERSON) {
        person_count++;
        num_rects++;
      }
      /* Conditions that user needs to set to encode the detected objects of
       * interest. Here, by default all the detected objects are encoded.
       * For demonstration, we will encode the first object in the frame. */
      if ((obj_meta->class_id == PGIE_CLASS_ID_PERSON
              || obj_meta->class_id == PGIE_CLASS_ID_VEHICLE)
          && num_rects == 1) {
        NvDsObjEncUsrArgs objData = { 0 };
        /* To be set by user */
        objData.saveImg = save_img;
        objData.attachUsrMeta = attach_user_meta;
        /* Set if Image scaling Required */
        objData.scaleImg = FALSE;
        objData.scaledWidth = 0;
        objData.scaledHeight = 0;
        /* Preset */
        objData.objNum = num_rects;
        /* Quality */
        objData.quality = 80;
        /*Main Function Call */
        nvds_obj_enc_process (ctx, &objData, ip_surf, obj_meta, frame_meta);
      }
    }
  }
  nvds_obj_enc_finish (ctx);
  frame_count++;
  return GST_PAD_PROBE_OK;
}
```

## 第二步，檢查usrMetaData是否的meta_type是不是NVDS_CROP_IMAGE_META
如果發現是NVDS_CROP_IMAGE_META，就儲存照片
```c
/* osd_sink_pad_buffer_probe will extract metadata received on OSD sink pad
 * and update params for drawing rectangle, object information. We also iterate
 * through the user meta of type "NVDS_CROP_IMAGE_META" to find image crop meta
 * and demonstrate how to access it.*/
static GstPadProbeReturn
osd_sink_pad_buffer_probe (GstPad * pad, GstPadProbeInfo * info,
    gpointer u_data)
{
  GstBuffer *buf = (GstBuffer *) info->data;

  guint num_rects = 0;
  NvDsObjectMeta *obj_meta = NULL;
  guint vehicle_count = 0;
  guint person_count = 0;
  NvDsMetaList *l_frame = NULL;
  NvDsMetaList *l_obj = NULL;
  NvDsDisplayMeta *display_meta = NULL;
  NvDsBatchMeta *batch_meta = gst_buffer_get_nvds_batch_meta (buf);
  g_print ("Running osd_sink_pad_buffer_probe...\n");
  for (l_frame = batch_meta->frame_meta_list; l_frame != NULL;
      l_frame = l_frame->next) {
    NvDsFrameMeta *frame_meta = (NvDsFrameMeta *) (l_frame->data);
    int offset = 0;
    /* To verify  encoded metadata of cropped frames, we iterate through the
    * user metadata of each frame and if a metadata of the type
    * 'NVDS_CROP_IMAGE_META' is found then we write that to a file as
    * implemented below.
    */
    char fileFrameNameString[FILE_NAME_SIZE];
    const char *osd_string = "OSD";

    /* For Demonstration Purposes we are writing metadata to jpeg images of
      * the first 10 frames only.
      * The files generated have an 'OSD' prefix. */
    if (frame_number < 11) {
      NvDsUserMetaList *usrMetaList = frame_meta->frame_user_meta_list;
      FILE *file;
      int stream_num = 0;
      while (usrMetaList != NULL) {
        NvDsUserMeta *usrMetaData = (NvDsUserMeta *) usrMetaList->data;
        if (usrMetaData->base_meta.meta_type == NVDS_CROP_IMAGE_META) {
          snprintf (fileFrameNameString, FILE_NAME_SIZE, "%s_frame_%d_%d.jpg",
              osd_string, frame_number, stream_num++);
          NvDsObjEncOutParams *enc_jpeg_image =
              (NvDsObjEncOutParams *) usrMetaData->user_meta_data;
          /* Write to File */
          file = fopen (fileFrameNameString, "wb");
          fwrite (enc_jpeg_image->outBuffer, sizeof (uint8_t),
              enc_jpeg_image->outLen, file);
          fclose (file);
        }
        usrMetaList = usrMetaList->next;
      }
    }
    for (l_obj = frame_meta->obj_meta_list; l_obj != NULL; l_obj = l_obj->next) {
      obj_meta = (NvDsObjectMeta *) (l_obj->data);
      if (obj_meta->class_id == PGIE_CLASS_ID_VEHICLE) {
        vehicle_count++;
        num_rects++;
      }
      if (obj_meta->class_id == PGIE_CLASS_ID_PERSON) {
        person_count++;
        num_rects++;
      }
      /* To verify  encoded metadata of cropped objects, we iterate through the
       * user metadata of each object and if a metadata of the type
       * 'NVDS_CROP_IMAGE_META' is found then we write that to a file as
       * implemented below.
       */
      char fileObjNameString[FILE_NAME_SIZE];

      /* For Demonstration Purposes we are writing metadata to jpeg images of
       * vehicles or persons for the first 100 frames only.
       * The files generated have a 'OSD' prefix. */
      if (frame_number < 100 && (obj_meta->class_id == PGIE_CLASS_ID_PERSON
              || obj_meta->class_id == PGIE_CLASS_ID_VEHICLE)) {
        NvDsUserMetaList *usrMetaList = obj_meta->obj_user_meta_list;
        FILE *file;
        while (usrMetaList != NULL) {
          NvDsUserMeta *usrMetaData = (NvDsUserMeta *) usrMetaList->data;
          if (usrMetaData->base_meta.meta_type == NVDS_CROP_IMAGE_META) {
            NvDsObjEncOutParams *enc_jpeg_image =
                (NvDsObjEncOutParams *) usrMetaData->user_meta_data;

            snprintf (fileObjNameString, FILE_NAME_SIZE, "%s_%d_%d_%d_%s.jpg",
                osd_string, frame_number, frame_meta->batch_id, num_rects,
                obj_meta->obj_label);
            /* Write to File */
            file = fopen (fileObjNameString, "wb");
            fwrite (enc_jpeg_image->outBuffer, sizeof (uint8_t),
                enc_jpeg_image->outLen, file);
            fclose (file);
            usrMetaList = NULL;
          } else {
            usrMetaList = usrMetaList->next;
          }
        }
      }
    }
    display_meta = nvds_acquire_display_meta_from_pool (batch_meta);
    NvOSD_TextParams *txt_params = &display_meta->text_params[0];
    txt_params->display_text = g_malloc0 (MAX_DISPLAY_LEN);
    offset =
        snprintf (txt_params->display_text, MAX_DISPLAY_LEN, "Person = %d ",
        person_count);
    offset =
        snprintf (txt_params->display_text + offset, MAX_DISPLAY_LEN,
        "Vehicle = %d ", vehicle_count);

    /* Now set the offsets where the string should appear */
    txt_params->x_offset = 10;
    txt_params->y_offset = 12;

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

    nvds_add_display_meta_to_frame (frame_meta, display_meta);
  }
  g_print ("Frame Number = %d Number of objects = %d "
      "Vehicle Count = %d Person Count = %d\n",
      frame_number, num_rects, vehicle_count, person_count);
  frame_number++;
  return GST_PAD_PROBE_OK;
}
```

# 加入自己客製的的metadata
參考deepstream-user-metadata-test範例的nvinfer_src_pad_buffer_probe
1. 有四個東西需要使用者自行提供
   1. user_meta_data : pointer to User specific meta data
   2. meta_type : Metadata type that user sets to identify its metadata
   3. copy_func : Metadata copy or transform function to be provided when there is buffer transformation
   4. release_func : Metadata release function to be provided when it is no longer required.


2. 這個範例添加一個亂數到metadata上面，以下是要達成這個目標要準備的函式
   1. user_meta_data
```c
void *set_metadata_ptr()
{
  int i = 0;
  gchar *user_metadata = (gchar*)g_malloc0(USER_ARRAY_SIZE);

  g_print("\n**************** Setting user metadata array of 16 on nvinfer src pad\n");
  for(i = 0; i < USER_ARRAY_SIZE; i++) {
    user_metadata[i] = rand() % 255;
    g_print("user_meta_data [%d] = %d\n", i, user_metadata[i]);
  }
  return (void *)user_metadata;
}   
```

   2. meta_type
記得要在在probe function裡面定義變數
```c
/** set the user metadata type */
#define NVDS_USER_FRAME_META_EXAMPLE (nvds_get_user_meta_type("NVIDIA.NVINFER.USER_META"))
NvDsMetaType user_meta_type = NVDS_USER_FRAME_META_EXAMPLE;
```

   3. copy_func

```c
/* copy function set by user. "data" holds a pointer to NvDsUserMeta*/
static gpointer copy_user_meta(gpointer data, gpointer user_data)
{
  NvDsUserMeta *user_meta = (NvDsUserMeta *)data;
  gchar *src_user_metadata = (gchar*)user_meta->user_meta_data;
  gchar *dst_user_metadata = (gchar*)g_malloc0(USER_ARRAY_SIZE);
  memcpy(dst_user_metadata, src_user_metadata, USER_ARRAY_SIZE);
  return (gpointer)dst_user_metadata;
}
```

   4. release_func

```c
/* release function set by user. "data" holds a pointer to NvDsUserMeta*/
static void release_user_meta(gpointer data, gpointer user_data)
{
  NvDsUserMeta *user_meta = (NvDsUserMeta *) data;
  if(user_meta->user_meta_data) {
    g_free(user_meta->user_meta_data);
    user_meta->user_meta_data = NULL;
  }
}

```

3. 新增一個probe把資料放入metadata
```c
/* Set nvds user metadata at frame level. User need to set 4 parameters after
 * acquring user meta from pool using nvds_acquire_user_meta_from_pool().
 *
 * Below parameters are required to be set.
 * 1. user_meta_data : pointer to User specific meta data
 * 2. meta_type: Metadata type that user sets to identify its metadata
 * 3. copy_func: Metadata copy or transform function to be provided when there
 *               is buffer transformation
 * 4. release_func: Metadata release function to be provided when it is no
 *                  longer required.
 *
 * osd_sink_pad_buffer_probe  will extract metadata received on OSD sink pad
 * and update params for drawing rectangle, object information etc. */

static GstPadProbeReturn
nvinfer_src_pad_buffer_probe (GstPad * pad, GstPadProbeInfo * info,
    gpointer u_data)
{
  GstBuffer *buf = (GstBuffer *) info->data;
  NvDsMetaList * l_frame = NULL;
  NvDsUserMeta *user_meta = NULL;
  NvDsMetaType user_meta_type = NVDS_USER_FRAME_META_EXAMPLE;

  NvDsBatchMeta *batch_meta = gst_buffer_get_nvds_batch_meta (buf);

    for (l_frame = batch_meta->frame_meta_list; l_frame != NULL;
      l_frame = l_frame->next) {
        NvDsFrameMeta *frame_meta = (NvDsFrameMeta *) (l_frame->data);

        /* Acquire NvDsUserMeta user meta from pool */
        user_meta = nvds_acquire_user_meta_from_pool(batch_meta);

        /* Set NvDsUserMeta below */
        user_meta->user_meta_data = (void *)set_metadata_ptr();
        user_meta->base_meta.meta_type = user_meta_type;
        user_meta->base_meta.copy_func = (NvDsMetaCopyFunc)copy_user_meta;
        user_meta->base_meta.release_func = (NvDsMetaReleaseFunc)release_user_meta;

        /* We want to add NvDsUserMeta to frame level */
        nvds_add_user_meta_to_frame(frame_meta, user_meta);
    }
    return GST_PAD_PROBE_OK;
}
```


# 將客製化訊息傳換json以之後發送訊息
nvmsgconv的功能:  
利用`NVDS_EVENT_MSG_META`metadata來產生JSON格式的"DeepStream Schema" payload。
所產生的payload會以`NVDS_META_PAYLOAD`的型態儲存到buffer。
除了`NvDsEventMsgMeta`定義的常用訊號結構，使用者還可以自訂客製化物件並加到`NVDS_EVENT_MSG_META`metadata。要加入自定義訊息`NvDsEventMsgMeta`提供"extMsg"和"extMsgSize"欄位。使用者可以把自定義的structure指針assign給"extMsg"，並且在"extMsgSize"指令資料大小。


以deepstream-test4為例，在這裡message放入了客製化訊息NvDsVehicleObject和NvDsPersonObject，如果想要客製化自己的訊息就必須要自己定義。

自製自己的客製化訊息
可以參考
`/opt/nvidia/deepstream/deepstream-6.2/sources/libs/nvmsgconv/deepstream_schema/eventmsg_payload.cpp`參考客製化訊息如何定義轉換成json

nvmsgconv的原始碼
/opt/nvidia/deepstream/deepstream-6.2/sources/gst-plugins/gst-nvmsgconv
/opt/nvidia/deepstream/deepstream-6.2/sources/libs/nvmsgconv

* nvmsgconv開啟除錯功能
debug-payload-dir : Absolute path of the directory to dump payloads for debugging

1. 以deepstream-test4為例，首先將模型的偵測結果`NvDsObjectMeta`轉換成`NvDsEventMsgMeta`，在這步將訊息struct加到`extMsg`上
2. 將製作好的`NvDsEventMsgMeta`加進buffer裡面，metadata為`NvDsUserMeta`，在這一步也要指定meta_copy_func、meta_free_func


# mvmsgbroker使用方法
以下將以rabbitmq為範例
1. 安裝rabbitmq client
說明文件在`/opt/nvidia/deepstream/deepstream/sources/libs/amqp_protocol_adaptor`的readme.md
```bash
 git clone -b v0.8.0  --recursive https://github.com/alanxz/rabbitmq-c.git
 cd rabbitmq-c
 mkdir build && cd build
 cmake ..
 cmake --build .
 sudo cp librabbitmq/librabbitmq.so.4 /opt/nvidia/deepstream/deepstream/lib/
 sudo ldconfig
```

2. 安裝rabbitmq server  

```bash
#Install rabbitmq on your ubuntu system: https://www.rabbitmq.com/install-debian.html
#The “Using rabbitmq.com APT Repository” procedure is known to work well

 sudo apt-get install rabbitmq-server

#Ensure rabbitmq service has started by running (should be the case):
 sudo service rabbitmq-server status

#Otherwise
 sudo service rabbitmq-server start
```

3. 設定連線詳細資訊
  1. 建立`cfg_amqp.txt`連線資訊檔(/opt/nvidia/deepstream/deepstream/sources/libs/amqp_protocol_adaptor 有範例)，並且傳給nvmsgbroker。內容範例如下

```bash
[message-broker]
hostname = localhost
port = 5672
username = guest
password = guest
exchange = amq.topic
topic = topicname
amqp-framesize = 131072
#share-connection = 1
```
  * exchange: 預設的exchange是`amq.topic`，可以更改成其他的
  * Topic : 設定要發送的topic名稱
  * share-connection : Uncommenting this field signifies that the connection created can be shared with other components within the same process.
  
  1. 直接將連線資訊傳給`msgapi_connect_ptr`

```c
 conn_handle = msgapi_connect_ptr((char *)"url;port;username;password",(nvds_msgapi_connect_cb_t) sample_msgapi_connect_cb, (char *)CFG_FILE);
```

4. 測試用程式
在`/opt/nvidia/deepstream/deepstream/sources/libs/amqp_protocol_adaptor`有測試用的程式`test_amqp_proto_async.c`和`test_amqp_proto_sync.c`，可以用來測試連線是否成功，編譯方式如下
```bash
 make -f Makefile.test
 ./test_amqp_proto_async
 ./test_amqp_proto_sync
```
注意:
* 你可能須要root權限才能在這個資料夾編譯程式
* libnvds_amqp_proto.so 位於 /opt/nvidia/deepstream/deepstream-<version>/lib/


5. 測試和驗證發送出去的訊息
    * 建立exchange , queue，並且將他們綁定在一起  

```bash
# Rabbitmq management:
It comes with a command line tool which you can use to create/configure all of your queues/exchanges/etc
https://www.rabbitmq.com/management.html

# Install rabbitmq management plugin:
sudo rabbitmq-plugins enable rabbitmq_management

# Use the default exchange amq.topic
OR create an exchange as below, the same name as the one you specify within the cfg_amqp.txt
#sudo rabbitmqadmin -u guest -p guest -V / declare exchange name=myexchange type=topic

# Create a Queue
sudo rabbitmqadmin -u guest -p guest -V / declare queue name=myqueue durable=false auto_delete=true

#Bind Queue to exchange with routhing_key specification
rabbitmqadmin -u guest -p guest -V / declare binding source=amq.topic destination=myqueue routing_key=topicname

#To check if the queues were actually created, execute:
$ sudo rabbitmqctl list_queues
Listing queues
myqueue      0
```

    * 接收訊息
```bash
#Install the amqp-tools
sudo apt-get install amqp-tools

cat <<EOF > test_amqp_recv.sh
while read -r line; do
    echo "\$line"
done
EOF

chmod +x test_amqp_recv.sh
```

    * 執行consumer
```bash
amqp-consume  -q "myqueue" -r "topicname" -e "amq.topic" ./test_amqp_recv.sh
```

# 混用c 和 c++ 程式
https://hackmd.io/@rhythm/HyOxzDkmD  
https://embeddedartistry.com/blog/2017/05/01/mixing-c-and-c-extern-c/  


# async property
某些狀況下async property 設為true會讓pipeline卡住，還需要進一步了解原因

# nvmsgconv 詳細payload設定
在`/opt/nvidia/deepstream/deepstream/sources/libs/nvmsgconv/nvmsgconv.cpp`裡面可以看到`nvds_msg2p_ctx_create`這個函式，是用來產出payload的函式。在nvmsgconv讀取的yaml檔裡面可以設定的group和屬性如下

## sensor
* enable : 是否啟用這個sensor
* id : 對應NvDsEventMsgMeta的sensorId
* type : 
* description
* location
  * lat;lon;alt的格式
* coordinate
  * x;y;z的格式

## place

## analytics

## NvDsEventMsgMeta 轉換成json的詳細實作
在`/opt/nvidia/deepstream/deepstream-6.2/sources/libs/nvmsgconv/deepstream_schema/eventmsg_payload.cpp`這個程式裡，分別有sensor, place, analytics的轉換實作

# 客製化nvmsgconv payload
如果要客製化payload的話，可以參考`/opt/nvidia/deepstream/deepstream-6.2/sources/libs/nvmsgconv/deepstream_schema/eventmsg_payload.cpp`裡面的實作，並且加入自己需要的客製化payload。首先將整個`/opt/nvidia/deepstream/deepstream-6.2/sources/libs/nvmsgconv`複製到其他資料夾並且準備編譯環境

## 編譯環境
這裡介紹在Ubuntu20.04的桌上型主機上環境的配置方法，Jetson的環境配置方法可能略有不同。
* 下載並且編譯protobuf
在Ubuntu20.04下使用apt-get install protobuf 只會安裝到protobuf 3.6的版本，而許多標頭檔要到3.7以上才有，而且不能超過3.19，以免某些標頭檔又遺失。如果中間有步驟做錯，只要還沒`make install`，建議直接刪除protobuf的資料夾，重新下載並且編譯。

首先直接從github下載protobuf原始碼
```bash
git clone https://github.com/protocolbuffers/protobuf.git
```
切換版本到v3.19.6，並且更新submodule。
```bash
cd protobuf
git submodule update --init --recursive
./autogen.sh
```

編譯並且安裝，`make check`過程中如果有錯誤，編譯出來的程式可能會有部分功能遺失。
```bash
./configure
make
make check
sudo make install
sudo ldconfig # refresh shared library cache.
```

## 編譯客製化的nvmsgconv
接下來進到nvmsgconv的資料夾，修改一下最後產出的lib檔案名稱和install的位置，然後用`make`指令編譯


## 預訓練模型
/opt/nvidia/deepstream/deepstream-6.2/samples/models/tao_pretrained_mod
els/trafficcamnet


# usb相機
https://docs.nvidia.com/jetson/archives/r35.4.1/DeveloperGuide/text/SD/CameraDevelopment/CameraSoftwareDevelopmentSolution.html#applications-using-gstreamer-with-the-nvarguscamerasrc-plugin



## 儲存影像
https://forums.developer.nvidia.com/t/drawn-rectangle-is-not-available-on-encoded-frame/178022/7?u=jenhao


# 元件速度測量
https://forums.developer.nvidia.com/t/deepstream-sdk-faq/80236/12?u=jenhao  






參考:  
https://www.gclue.jp/2022/06/gstreamer.html  