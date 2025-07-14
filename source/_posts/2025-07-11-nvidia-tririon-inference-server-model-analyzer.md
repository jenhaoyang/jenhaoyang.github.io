---
title: nvidia-tririon-inference-server-model-analyzer
date: 2025-07-11 18:03:46
categories:
tags:
---


# 環境
Ubuntu 22.04
trition inference server version : 23.08 


在git clone下來的model_analyzer資料夾下啟動docker
```
docker run -it --gpus all \
      -v /var/run/docker.sock:/var/run/docker.sock \
       -v $(pwd)/examples/quick-start:$(pwd)/examples/quick-start \
       -v $(pwd)/examples/quick-start/output-model-repo:$(pwd)/examples/quick-start/output-model-repo \
       --net=host nvcr.io/nvidia/tritonserver:23.08-py3-sdk
```
假設現在所在路徑$(pwd)是/home/user/model_inference


啟動docker後會發現quick-start所在的的路徑會跟host主機`一模一樣`，因此下面的指令可以直接用跟host`一模一樣`的路徑去執行
```
model-analyzer profile \
    --model-repository  /home/user/model_inference/model_analyzer/examples/quick-start \
     --profile-models add_sub --triton-launch-mode=docker \
     --output-model-repository-path /home/user/model_inference/model_analyzer/examples/quick-start/output-model-repo/add \
     --export-path profile_results
```

# 查詢
https://github.com/triton-inference-server/model_analyzer/blob/5e3746f738b56118b31f28d9472db04f7361aaf8/docs/config_search.md#examples-of-additional-model-config-parameters

https://github.com/triton-inference-server/tutorials/blob/main/Conceptual_Guide/Part_4-inference_acceleration/README.md

onnx execution_accelerators?


# 參考:
https://github.com/triton-inference-server/model_analyzer/blob/4b45d2daeb9f574d13ae0e774677c87c04ef2124/docs/quick_start.md