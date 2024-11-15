---
layout: post
title: Python OpenCV圖片裁剪
date: 2022-08-17 17:48 +0800
categories: [機器視覺]
tags: [opencv]
---

# OpenCV 座標

# Python OpenCV 裁剪圖片
OpenCV 的蒲覑本質上就是一個 numpy array， 因此可以利用numpy array提供的方法來完成裁剪圖片的功能
```
crop_img = img[y:y+h, x:x+w] # x, y 為圖片的裁剪圖片的左上角座標，h, w 分別為裁剪圖片的高和寬
```

參考:  
https://stackoverflow.com/a/15589825