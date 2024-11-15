---
layout: post
title: Dive into DeepLearning
date: 2023-07-05 16:50 +0800
categories: [深度學習]
tags: [book]
pin: true
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

### 7.2.1 The Cross-Correlation Operation
經過Cross-Correlation Operation後，輸出的tensor尺寸為
$(n_h − k_h + 1 ) × (n_w − k_w + 1 )$

### 7.2.2 Convolutional Layers
Convolutional Layers就是經過Cross-Correlation Operation之後的tensor對每一個element都加上一個bias。Conv的kernel如同前面MLP的權重，初始化的時候我們是用亂數初始化
### 7.2.3 Object Edge Detection in Images
已知一個人工製作的邊緣偵測器kernel為[1, -1]可以偵測垂直線，等一下會嘗試讓電腦自己學習出這個kernel
### 7.2.4 Learning a Kernel
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


### 7.2.5 Cross-Correlation and Convolution
再Deeplearning 使用的Convolution計算方式其實真正的名稱是Cross-Correlation，但是由於Cross-Correlation和Convolution的差別只在於kernerl上下和左右都互換，但對於traing的結果影響不大，所以依然稱為Convolution

### 7.2.6 Feature Map and Receptive Field
convolutional layer的輸出稱為Feature Map。  
receptive field則是影響輸出結果之前的所有元素。()


## 7.3 Padding and Stride
單純使用conv會使得後面layer的input快速變小。Padding可以解決這個狀況，相反的Stride則是用來快速讓輸入變小。
### 7.3.1 Padding
padding和kernel關係的公式如下，假設row padding $p_h$, column padding $p_w$:
$$(n_h − k_h + p_h + 1 ) × (n_w − k_w + p_w + 1 )$$
我們可以藉由$p_h=k_h-1$和$p_w=k_w-1$讓輸入和輸出的長寬一樣。  
如果$k$為奇數，則padding剛好可以填充到兩側，如果$k$為偶數則兩側的padding數量有一邊會多一個。

### 7.3.2 Stride
當高的stride為$s_h$寬的stride為$s_w$，則輸出為
$$\lfloor(n_h-k_h+p_h+s_h)/s_h\rfloor \times \lfloor(n_w-k_w+p_w+s_w)/s_w\rfloor$$

## 7.4 Multiple Input and Multiple Output Channels
### 7.4.1 Multiple Input Channels
當輸入的channel數量大於1的時候，每一個channel會需要至少一個kernel。假設現在輸入有3個channel，每一個channel分別對一個kernel之後相加結果就是輸出。
```python
def corr2d_multi_in(X, K):
    # Iterate through the 0th dimension (channel) of K first, then add them up
    return sum(d2l.corr2d(x, k) for x, k in zip(X, K))
X = torch.tensor([[[0.0, 1.0, 2.0], [3.0, 4.0, 5.0], [6.0, 7.0, 8.0]],
[[1.0, 2.0, 3.0], [4.0, 5.0, 6.0], [7.0, 8.0, 9.0]]])
K = torch.tensor([[[0.0, 1.0], [2.0, 3.0]], [[1.0, 2.0], [3.0, 4.0]]])
corr2d_multi_in(X, K)
# tensor([[ 56., 72.],
# [104., 120.]])
```
### 7.4.2 Multiple Output Channels
為了讓神經網路學到更多feature，我們嘗試讓同一層conv layer的kernel為三維而且第三個維度和輸入的channel數一樣$c_i\times k_h \times k_w$，例如輸入為3個channel，我們就設計kernel的第三個維度為3。由於每一組kernel最後的輸出都是一個channel。所以最後輸出的channel數就是kernel的組數。因此 convolution layer的維度就變成$c_o \times c_i \times k_h \times k_w$

* `stack()`在這裡會把沿著指定的dimension把tensor堆疊
```python
def corr2d_multi_in_out(X, K):
    # Iterate through the 0th dimension of K, and each time, perform
    # cross-correlation operations with input X. All of the results are
    # stacked together
    return torch.stack([corr2d_multi_in(X, k) for k in K], 0)
K = torch.stack((K, K + 1, K + 2), 0)
K.shape
#torch.Size([3, 2, 2, 2])
corr2d_multi_in_out(X, K)
# tensor([[[ 56., 72.],
#         [104., 120.]],
#         [[ 76., 100.],
#         [148., 172.]],
#         [[ 96., 128.],
#         [192., 224.]]])
```

### 7.4.3 1 × 1 Convolutional Layer
1 × 1 Convolutional Layer唯一能夠影像的就是channel這個維度。
1 × 1 Convolutional Layer可以視為在每一個單獨的像素位置上的channel的fully conection layer。並且將channel數由$c_i$轉換成$c_o$。

## 7.5 Pooling
Pooling讓我們可以專注在影像比較大的區域而不會因為影像中細微的變化就影響輸出結果。
### 7.5.1 Maximum Pooling and Average Pooling
池化層沒有任何參數，將window所有元素平均為Average Pooling，取出最大值則為Maximum Pooling。

### 7.5.2 Padding and Stride
跟conv layer一樣pooling也有padding和Stride，而由於pooling主要目的是從一個區域擷取資訊，所以深度學習框架預設將Stride的大小設定跟pooling window一樣大，不過我們還是可以自行修改大小。

### 7.5.3 Multiple Channels
不同於conv layer，pooling對每一個channel是分開處理的，而不像conv layer會把其他channel的結果相加。因此pooling的輸入和輸出channel數目是一樣的。

## 7.6 Convolutional Neural Networks (LeNet)
接下來介紹LeNet

### 7.6.1 LeNet
下面是LeNet的Pytorch實作，這裡使用了`Xavier initialization`
```python
def init_cnn(module): #@save
    """Initialize weights for CNNs."""
    if type(module) == nn.Linear or type(module) == nn.Conv2d:
    nn.init.xavier_uniform_(module.weight)

class LeNet(d2l.Classifier): #@save
    """The LeNet-5 model."""
    def __init__(self, lr=0.1, num_classes=10):
        super().__init__()
        self.save_hyperparameters()
        self.net = nn.Sequential(
            nn.LazyConv2d(6, kernel_size=5, padding=2), nn.Sigmoid(),
            nn.AvgPool2d(kernel_size=2, stride=2),
            nn.LazyConv2d(16, kernel_size=5), nn.Sigmoid(),
            nn.AvgPool2d(kernel_size=2, stride=2),
            nn.Flatten(),
            nn.LazyLinear(120), nn.Sigmoid(),
            nn.LazyLinear(84), nn.Sigmoid(),
            nn.LazyLinear(num_classes))
```

### 7.6.2 Training
雖然CNN的參數比較少，但是計算量並不少，用GPU計算更適合。

# 8 Modern Convolutional Neural Networks
## 8.1 Deep Convolutional Neural Networks (AlexNet)
硬體運算速度的提升增加了CNN可行性。

### 8.1.1 Representation Learning
有一派的研究員相信圖片中的feature是可以讓電腦自己學習得到的。AlexNet在地層layer所學到的kernel跟傳統機器學習手工製作的feature很像。  
推動CNN兩大因素別是"資料"和"電腦硬體"，過去的資料集都很小而且電腦運算速度很慢。
### 8.1.2 AlenNet
AlexNet把sigmoid activation function改成ReLU activtion function。  
ReLU activation function，讓模型訓練變得更加容易。因為如果初始化的參數很接近0或1，sigmoid activation function的輸出值會很接近0，這使的反向傳播幾乎沒有作用。而ReLU activation function卻不會有這種情形。  
AlexNet在訓練的時候也大量的利用圖片擴增的技巧，這使得訓練結果不會overfitting。  

### 8.1.4 Discussion
AlexNet弱點是他最後的兩個layer佔用了非常大量的記憶體和運算，這對需要高速運算的場景非常不利。
另外一個可以注意的點是即使AlexNet的參數量遠大於資料及照片總數，AlexNet也幾乎沒有Overfitting。這是因為有用到現代化的正規化方法如Dropout。

## 8.2 Networks Using Blocks (VGG)
神經網絡架構的設計日益抽象化，研究人員從思考個別神經元逐漸轉向整個層面，然後到塊狀結構，即層的重複模式。十年後，這已經進一步發展到研究人員使用完整的訓練模型，將它們重新應用於不同但相關的任務。這種大型 pretrained models通常被稱為foundation models 。  
這種Blocks的概念最早出現在VGG network，使用迴圈和子程序，可以在任何現代深度學習框架中輕鬆地在代碼中實現這些重複的結構。

### 8.2.1 VGG Blocks
CNNs的基本構建塊是以下順序的序列：（i）帶填充的卷積層以保持解析度，（ii）如ReLU之類的非線性激活函數，（iii）如最大池化之類的池化層以減小解析度。這種方法的一個問題是空間解析度下降得相當迅速。特別是，這在所有維度（d）用完之前，對於網絡在卷積層上存在著 $log_2d$的硬限制。例如，在ImageNet的情況下，以這種方式不可能有超過8個卷積層。  
Simonyan和Zisserman（2014）的關鍵想法是使用連續的小卷層來取代一個大卷積層，例如兩個3x3的卷積層其實涵蓋的像素跟一個5x5一樣大，經過觀察利用連續小的卷積層的效果跟直接用一個大卷積層的效果相似。  
堆疊3×3的卷積後來成為後來的深度網絡的黃金標準，直到最近由Liu等人（2022）重新審查了這個設計決策。
* VGG的構造:一連串的$3 \times 3$kernel且padding為1的conv layer最後接著一個$2 \times 2$ stride為2的max-pooling。下面程式實作一個VGG block，以$3 \times 3$kernel的數量num_convs和輸出channel數量out_channels作為參數。

```python
def vgg_block(num_convs, out_channels):
    layers = []
    for _ in range(num_convs):
        layers.append(nn.LazyConv2d(out_channels, kernel_size=3, padding=1))
        layers.append(nn.ReLU())
    layers.append(nn.MaxPool2d(kernel_size=2,stride=2))
    return nn.Sequential(*layers)
```

### 8.2.2 VGG Network
VGG Network可以拆成兩個部分，前半段是多個conv layer和pooling layer組成，後半段是fully connected layer所組成。

下面程式定義了VGG network，在這裡我們利用for迴圈將多個VGG block組合成VGG netowrk
```python
class VGG(d2l.Classifier):
    def __init__(self, arch, lr=0.1, num_classes=10):
        super().__init__()
        self.save_hyperparameters()
        conv_blks = []
        for (num_convs, out_channels) in arch:
            conv_blks.append(vgg_block(num_convs, out_channels))
        self.net = nn.Sequential(
            *conv_blks, nn.Flatten(),
            nn.LazyLinear(4096), nn.ReLU(), nn.Dropout(0.5),
            nn.LazyLinear(4096), nn.ReLU(), nn.Dropout(0.5),
            nn.LazyLinear(num_classes))
        self.net.apply(d2l.init_cnn)
```

## 8.3 Network in Network (NiN)
VGG, LeNet, VGG都有一個共通的缺點。
1. fully connected layer有非常大的參數量。
2. fully connected layer沒辦法放在模型最後面以外的地方。
Network in Network解決了上面兩個問題。他利用了像面兩個方式解決。
1. 利用$1 \times 1$ conv layer來取代fully connected layer
2. 在模型的最後面使用global average pooling
NiN的好處是沒有fully connected layer因此參數量大大減少。
### 8.3.1 NiN Blocks
由一個conv layer加上兩個$1 \times 1$ conv layer組成


## 8.4 Multi-Branch Networks (GoogLeNet)
GoogLeNet可以說是第一個將模型拆解成三個部位stem, body, head的模型。

### 8.4.1 Inception Blocks
Inception Blocks跟前面模型不一樣的是一個block有三個分支

## 8.5 Batch Normalization
Batch Normalization是一個能夠有效加速深度模型收斂的方法，與 residual blocks可以讓模型深度達到一百層。

### 8.5.1 Training Deep Networks
Batch normalization 可以被套用在單一個layer或是所有layer，如此一來對每一層的input都先做一次 normalization(也就是平均為0標準差為1)。
注意到如果我們套用batch normalization在minibatches為一的狀況下，我們不會學到任何東西，因為所有的 hidden unit都會變成0。  
因此使用Batch normalization的時候batch size的大小變成十分重要。  
Batch normalization中scale parameter 和shift parameter是透過模型學習而得。

### 8.5.2 Batch Normalization Layers




# CH9 Recurrent Neural Network(RNN)
在這之前我們的資料長度都是固定的，在這章將要學習如何讓模型學習長度不固定的序列資料。
RNN利用recurrent connection讓模型可以學習序列化的資料，可以把他想成一個迴授迴路

9.1 序列化資料
在此之前我們的特徵向量長度是固定的，而序列化資料的特徵向量是以時間排序而且長度不固定的資料。

