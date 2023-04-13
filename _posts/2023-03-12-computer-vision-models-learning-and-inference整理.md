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

## maximum likelihood
maximum likelihood目標是要找到一組參數$$\hat{\theta}$$使得資料$$\{x_i\}^I_{i=1}$$出現的機率最大化。而計算likelihood function 在單一個資料點$$x_i$$，也就是$$Pr(x_i|\theta)$$，我們只需在$$x_i$$處評估概率密度函數。
假設每一個資料點都是從分布中獨立被取出的，那麼所有資料點的機率$$Pr(x_{i...I}|{\theta})$$就是所有單獨資料點代入likelihood function的乘積。因此要maximum likelihood可以寫成  

$$\hat{\theta} = argmax_{\theta}[Pr(x_{i...I}|{\theta})] =argmax_{\theta} [\prod_{i=1}^I Pr(x_i|\theta)]$$

式子中的$$argmax_{\theta}f[\theta]$$是指找到一組參數組$${\theta}$$，讓$$argmax_{\theta}f[\theta]$$最大化。

要計算新資料$$x^*$$的predictive distribution，只需將新資料和我們找到的參數組帶入likelihood function，計算出機率即可。


## log likehood
log likehood好處
https://bookdown.org/dereksonderegger/571/13-maximum-likelihood-estimation.html#likelihood-function
