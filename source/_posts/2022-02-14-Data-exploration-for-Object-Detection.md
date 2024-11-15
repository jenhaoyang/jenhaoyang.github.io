---
layout: post
title: Data exploration for Object Detection
date: 2023-02-14 14:45 +0800
categories: [深度學習]
tags: [train_model]     # TAG names should always be lowercase
---

# 資料整體的品質
首先可以對整個資料集做一個初步的檢查，包含
1. 瀏覽整份資料集
2. 確認沒有嚴重有誤的照片(例如全黑的照片)
3. 確認所有照片都能夠被電腦讀取(以免訓練到一半程式被中斷)

# 照片尺寸和深寬比
對於整份資料集，統計所有照片的深寬比和尺寸十分重要，這些將會影響anchor size 和 ratios。通常有三種情況
1. 照片大小和深寬比都一樣: 
   只需要決定縮放比例
2. 照片大小和深寬比不同但差異不大，深寬比都介於0.7~1.5: 
   可以non-destructive resize(也就是不改變深寬比的縮放) -> padding
3. 照片大小和深寬比差異很大



參考:  
https://neptune.ai/blog/data-exploration-for-image-segmentation-and-object-detection  