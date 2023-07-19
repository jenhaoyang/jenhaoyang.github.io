---
layout: post
title: Dive into DeepLearning
date: 2023-07-05 16:50 +0800
categories: [深度學習]
tags: [book]
---
# 數學符號說明
$z = g \circ f $就是z(x) = g(f(x))，也就是$y = f(x), z = g(y)$

https://math.stackexchange.com/a/1092727
$x \in R$ : x是一個(一維)實數純量，例如
$x =-2$ 或 $x=42$
$x \in R^{n*d}$
# Ch2
## 2.5 Automatic Diff erentiation
現代的深度學習都有提供automatic differentiation(autograd)和backpropagation的功能，下面將介紹如何使用

### 2.5.2 Backward for Non-Scalar Variables
為什麼要先sum()之後才backpropagation?

cs231n
http://cs231n.stanford.edu/handouts/derivatives.pdf
* Scalar in Scalar out: 利用chain rule就可以計算backpropagation當函式為$f, g : R \rightarrow R$且$z = (g \circ f)(x)$則$\frac{\partial z}{\partial x}=\frac{\partial z}{\partial y}\frac{\partial y}{\partial x}$。這告訴我們我們移動x一點點$\Delta_x$則y移動的量如下$$\Delta_y=\frac{\partial y}{\partial x}\Delta_x$$，而z移動的量為$$\frac{\partial z}{\partial y}\Delta_y=\frac{\partial z}{\partial y}\frac{\partial y}{\partial x}\Delta_x$$
  
* Vector in, scalar out: 利用Gradient可以計算backpropagation
當函式為$f : R^N \rightarrow R$且$x \in R^N$也就是說x是一個vector，而$\nabla_xf(x) \in R^N$，也就是Gradient的結果也是一個vector。繼續利用前面chain rule的概念$$x\rightarrow x+\Delta_x \Rightarrow y \rightarrow \approx y+\frac{\partial y}{\partial x} \cdot \Delta_x$$不過現在的狀況$x$, $\Delta_x$, \frac{\partial y}{\partial x} 都是vector，而兩個vector的內積剛好是scalar

* Vector in, Vector out
現在函式$f:R^N \rightarrow R^M$

* Jacobian
linear map
matrix代表了linear map，而determinant代表linear map之後面積放大的倍率，而負的determinant代表座標方向反轉
1. 微分不應該單純的想成斜率，積分也不應該單純想成面積。而是用另以種角度來想，微分代表在某一個點上，把它放很大來看之後他的線性映射是如何(linear map)

2. 對二維函式來說，Jacobian matrix就是在(a, b)附近的linear map



https://youtu.be/wCZ1VEmVjVo   
https://youtu.be/CfW845LNObM  
https://angeloyeo.github.io/2020/07/24/Jacobian_en.html  
# CH7

## 7.2.1 The Cross-Correlation Operation
經過Cross-Correlation Operation後，輸出的tensor尺寸為
$(n_h − k_h + 1 ) × (n_w − k_w + 1 )$

## 7.2.2 Convolutional Layers
Convolutional Layers就是經過Cross-Correlation Operation之後的tensor對每一個element都加上一個bias。Conv的kernel如同前面MLP的權重，初始化的時候我們是用亂數初始化
## 7.2.3 Object Edge Detection in Images
已知一個人工製作的邊緣偵測器kernel為[1, -1]可以偵測垂直線，等一下會嘗試讓電腦自己學習出這個kernel
## 7.2.4 Learning a Kernel
接下來我們要常識讓電腦自動學習kernel，為了簡單起見bias為0。(為什麼先sum再backward?)(https://www.youtube.com/watch?v=Q7KekwUricc)(https://dlvu.github.io/)
以下程式可以自動學習kernel

```python
# Construct a two-dimensional convolutional layer with 1 output channel and a
# kernel of shape (1, 2). For the sake of simplicity, we ignore the bias here

import torch
from torch import nn
from conv import corr2d

X = torch.tensor([[1., 1., 0., 0., 0., 0., 1., 1.],
                  [1., 1., 0., 0., 0., 0., 1., 1.],
                  [1., 1., 0., 0., 0., 0., 1., 1.],
                  [1., 1., 0., 0., 0., 0., 1., 1.],
                  [1., 1., 0., 0., 0., 0., 1., 1.],
                  [1., 1., 0., 0., 0., 0., 1., 1.]])

Y = torch.tensor([[ 0.,  1.,  0.,  0.,  0., -1.,  0.],
                  [ 0.,  1.,  0.,  0.,  0., -1.,  0.],
                  [ 0.,  1.,  0.,  0.,  0., -1.,  0.],
                  [ 0.,  1.,  0.,  0.,  0., -1.,  0.],
                  [ 0.,  1.,  0.,  0.,  0., -1.,  0.],
                  [ 0.,  1.,  0.,  0.,  0., -1.,  0.]])

X = X.reshape((1, 1, 6, 8))
Y = Y.reshape((1, 1, 6, 7))
lr = 3e-2 # Learning rate
conv2d = nn.LazyConv2d(1, kernel_size=(1, 2), bias=False)

for i in range(10):
    Y_hat = conv2d(X)
    l = (Y_hat - Y) ** 2
    conv2d.zero_grad()
    l.sum().backward()
    # Update the kernel
    conv2d.weight.data[:] -= lr * conv2d.weight.grad
    if (i + 1) % 2 == 0:
        print(f'epoch {i + 1}, loss {l.sum():.3f}')
print(conv2d.weight.data.reshape((1, 2)))
```
### PyTorch Lazy modules的用途: 
可以避免寫死in_features，對於動態的in_features來說比較方便
https://jarvislabs.ai/blogs/PyTorch-lazy-modules/


## 7.2.5 Cross-Correlation and Convolution
再Deeplearning 使用的Convolution計算方式其實真正的名稱是Cross-Correlation，但是由於Cross-Correlation和Convolution的差別只在於kernerl上下和左右都互換，但對於traing的結果影響不大，所以依然稱為Convolution

## 7.2.6 Feature Map and Receptive Field
convolutional layer的輸出稱為Feature Map。  
receptive field則是影響輸出結果之前的所有元素。()


# 7.3 Padding and Stride
單純使用conv會使得後面layer的input快速變小。Padding可以解決這個狀況，相反的Stride則是用來快速讓輸入變小。
## 7.3.1 Padding
padding和kernel關係的公式如下，假設row padding $p_h$, column padding $p_w$:
$$(n_h − k_h + p_h + 1 ) × (n_w − k_w + p_w + 1 )$$
我們可以藉由$p_h=k_h-1$和$p_w=k_w-1$讓輸入和輸出的長寬一樣。  
如果$k$為奇數，則padding剛好可以填充到兩側，如果$k$為偶數則兩側的padding數量有一邊會多一個。

## 7.3.2 Stride
當高的stride為$s_h$寬的stride為$s_w$，則輸出為
$$\lfloor(n_h-k_h+p_h+s_h)/s_h\rfloor \times \lfloor(n_w-k_w+p_w+s_w)/s_w\rfloor$$

# 7.4 Multiple Input and Multiple Output Channels
## 7.4.1 Multiple Input Channels
