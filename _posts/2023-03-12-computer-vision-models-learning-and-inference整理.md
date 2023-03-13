---
layout: post
title: Computer Vision Models Learning and Inference整理
date: 2023-03-12 21:47 +0800
---

# CH4 擬和機率模型
這章節的目標是將機率模型擬和到資料$$\{x_i\}^I_{i=1}$$，這個過程稱為`學習`learning，目標是找到模型的參數組$$\theta$$。此外我們也學習如何用學習後的模型對新的數據$$x^*$$計算出他的機率，這過程evaluating the predictive distribution。  
在這章我們探討三種方法
1. maximum likelihood
2. maximum a posteriori
3. Bayesian approach