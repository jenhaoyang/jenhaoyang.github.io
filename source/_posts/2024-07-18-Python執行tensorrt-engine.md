---
layout: post
title: Python執行tensorrt-engine
date: 2024-07-18 17:31 +0800
---

# 載入Region_TRT plugin
必須要載入libnvinfer_plugin，可以用python的trt.init_libnvinfer_plugins載入，
```python
TRT_LOGGER = trt.Logger(trt.Logger.ERROR)
trt.init_libnvinfer_plugins(TRT_LOGGER,"")
```

https://docs.nvidia.com/deeplearning/tensorrt/api/python_api/infer/Plugin/IPluginRegistry.html#tensorrt.init_libnvinfer_plugins


# 印出目前已經載入的plugin
PLUGIN_CREATORS = trt.get_plugin_registry().plugin_creator_list
for plugin_creator in PLUGIN_CREATORS:
    print(plugin_creator.name)

參考並修改:https://developer.nvidia.com/zh-cn/blog/tensorrt-custom-layer-cn/


# tensorrt Region layer 說明書
https://docs.nvidia.com/deeplearning/tensorrt/api/c_api/structnvinfer1_1_1plugin_1_1_region_parameters.html#ad2c6bba4f07221add9b6d9abf0a4e312


inference範例參考:
https://leimao.github.io/blog/TensorRT-Python-Inference/