---
layout: post
title: Dive into DeepLearning
date: 2023-07-05 16:50 +0800
categories: [深度學習]
tags: [book]
---

# 符號說明
https://math.stackexchange.com/a/1092727
$x \in R$ : x是一個(一維)實數純量，例如
$x =-2$ 或 $x=42$
$x \in R^{n*d}$

# CH7

## 7.2.1 The Cross-Correlation Operation
經過Cross-Correlation Operation後，輸出的tensor尺寸為
$(n_h − k_h + 1 ) × (n_w − k_w + 1 )$

## 7.2.2 Convolutional Layers
Convolutional Layers就是經過Cross-Correlation Operation之後的tensor對每一個element都加上一個bias。Conv的kernel如同前面MLP的權重，初始化的時候我們是用亂數初始化
## 7.2.3 Object Edge Detection in Images
已知一個人工製作的邊緣偵測器kernel為[1, -1]可以偵測垂直線，等一下會嘗試讓電腦自己學習出這個kernel
## 7.2.4 Learning a Kernel