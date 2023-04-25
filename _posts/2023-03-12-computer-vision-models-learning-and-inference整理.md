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

## Maximum Posteriori
在Maximum Posteriori（MAP）擬合中，我們引入有關參數$$\theta$$的先驗信息。由於我們可能對可能的參數值有所了解。例如，在時間序列中，時間 t 的參數值可以告訴我們在時間 t + 1 可能的值，於是將此信息將被編碼在先驗分佈中。  
如同它的名稱，Maximum Posteriori方法將會找到一組參數組$$\hat{\theta}$$，使得Posteriori probability $$Pr(\theta|x_{i...I})$$最大化。


$$\hat{\theta} = argmax_{\theta}[Pr(\theta|x_{i...I})]$$

$$ =argmax_{\theta} [\frac{Pr(x_{i...I}|\theta)Pr(\theta)}{Pr(x_{i...I})}]$$

$$ =argmax_{\theta} [\frac{\prod_{i=1}^I Pr(x_i|\theta)Pr(\theta)}{Pr(x_{i...I})}]$$

在這裡第一行可以由貝葉斯定理推得第二行。另外由於我們找的是參數組$$\theta$$的最大值，所以分母的常數$$Pr(x_{i...I})$$是可以忽略的。因此我們可以將式子簡化成

$$\hat{\theta} = argmax_{\theta} [\prod_{i=1}^I Pr(x_i|\theta)Pr(\theta)]$$


我們可以發現他其實跟maximum likelihood只差了一項先驗分佈$$Pr(\theta)$$，所以maximum likelihood其實是maximum posteriori的一種特例，也就是maximum likelihood是$$Pr(\theta)$$是一個常數的情況。

## Bayesian Approach
Bayesian Approach裡，我們不再把參數$$\theta$$當作一個常數，而是承認一件顯而易見的事實，參數組$$\theta$$可能不是唯一的。因此我們嘗試利用貝葉思定裡計算$$Pr(\theta|x_{i...I})$$，也就是在資料$$x_{i...I}$$出現的情況下，參數組$$\theta$$的機率分佈。

$$Pr(\theta|x_{i...I}) = \frac{\prod_{i=1}^I Pr(x_i|\theta)Pr(\theta)}{Pr(x_{i...I})}$$

而要驗證Bayesian Approach會比較複雜一點，因為跟前面不一樣我們沒有一個固定的參數組$$\theta$$可以帶入並且計算出機率，在這裡我們必須可能模型的機率分布。因此我們用以下方法計算。

$$Pr(x^*|x_{i...I}) = \int Pr(x^*|\theta)Pr(\theta|x_{i...I})d\theta$$

這條式子可以用以下方式解讀:  
$$Pr(x^*|\theta)$$ 是對於給定的參數組  $$\theta$$，$$x^*$$出現的機率，因此這個積分可以視為使用不同的參數$$\theta$$所做出預測的加權總和，其中權重是由參數的posterior probability distribution $$Pr(\theta|x_{i...I})$$ 來決定的（代表我們對不同參數正確性的信心程度）。

## 統一三種方法的 predictive density calculations
如果我們將maximum likelihood, maximum posteriori 的參數的機率分布視為一種特例，也就是maximum likelihood, maximum posteriori參數分布全部集中在$$\hat{\theta}$$。正式的說法就是參數分布為一個以$$\hat{\theta}$$為中心的delta function。一個delta function $$\delta[z]$$是一個函式他的積分為1，而且在除了中心點z以外的任何地方都是0。我們將剛才predictive density帶入delta function，可以得到以下結果。


$$Pr(x^*|x_{i...I}) = \int Pr(x^*|\theta)\delta[\theta - \hat{\theta}]d\theta$$

$$= Pr(x^*|\hat{\theta})$$


## 範例一
下面範例我們考慮擬和一個univariate normal model到一組數據$$\{x_i\}^I_{i=1}$$。
首先univariate normal model的probability density function為


$$Pr(x|\mu,\sigma^2) = Norm_x[\mu,\sigma^2] = \frac{1}{\sqrt{2\pi\sigma^2}}exp[-\frac{(x-\mu)^2}{2\sigma^2}]$$  


他有兩個參數平均$$\mu$$和變異數$$\sigma^2$$，首先我們從一個平均為1，變異數為1的normal distribution中取出$$I$$個數據$$x_{1...I}$$，我們的目標是利用前面的三種方法擬和抽取出來的數據集。

### 方法一Maximum likelihood estimation
對於觀測到的數據$Pr(x_{1...I}|\mu,\sigma^2)$$來說，參數 $$\{\mu,\sigma^2\}$$ 的概率(likelihood) Pr(x_1...I |µ, σ^2) 是通過對每個數據點分別評估概率密度函數，然後取乘積得到的。  

$$Pr(x_{1...I}|\mu,\sigma^2) = \prod_{i=1}^I Pr(x_i|\mu,\sigma^2)$$

$$= \prod_{i=1}^I Norm_{x_i}[\mu,\sigma^2]$$

$$= \frac{1}{\sqrt{2\pi\sigma^2}}exp[-\sum_{i=1}^I \frac{(x_i-\mu)^2}{2\sigma^2}]$$

很明顯的某些$$\{\mu,\sigma^2\}$$參數組會使得likelihood比其他參數組還高。而且我們可以在二維平面視覺化各種參數組的likelihood，我們將以平均$$\mu$$和變異數$$\sigma^2$$為軸，而Maximum likelihood的解答就是在圖形的頂點(圖4.2)。也就是以下式子的解答。


$$\hat{\mu}, {\hat\sigma}^2 = argmax_{\mu,\sigma^2} [Pr(x_{1...I}|\mu,\sigma^2)]$$

理論上我們可以藉由微分$$Pr(x_{1...I}|\mu,\sigma^2)$$來求解，但是實際上$$Pr(x_{1...I}|\mu,\sigma^2)$$太複雜，因此我們可以將$$Pr(x_{1...I}|\mu,\sigma^2)$$取log，因為log是一個單調遞增函數，經過轉換後的$$Pr(x_{1...I}|\mu,\sigma^2)$$的最大值會在相同的地方。代數上，對數把各個數據點的可能性的乘積轉化為它們的總和，因此可以將每個數據點的貢獻分離出來。於是Maximum likelihood可以用以下方式計算。

$$\hat{\mu}, {\hat\sigma}^2 = argmax_{\mu,\sigma^2} [\sum_{i=1}^I log [Norm_{x_{i}}[\hat{\mu}, {\hat\sigma}^2]]]$$

$$= argmax_{\mu,\sigma^2} [-0.5Ilog[2\pi] - 0.5Ilog(\sigma^2) - 0.5\sum_{i=1}^I\frac{(x_i-\mu)^2}{\sigma^2}]$$

接著對對 log likelihood function L 進行$$\mu$$的偏微分。
$$\frac{\partial L}{\partial \mu} = \sum_{i=1}^I \frac{(x_i-\mu)}{\sigma^2}$$
$$= \frac{\sum_{i=1}^I x_i}{\sigma^2} - \frac{I\mu}{\sigma^2} = 0$$

整理後可以得到
$$\hat{\mu} = \frac{\sum_{i=1}^I x_i}{I}$$

利用類似的方利用類似的方法可以得到變異數$$\sigma^2$$的解答為
$$\hat{\sigma}^2 = \frac{\sum_{i=1}^I (x_i-\hat{\mu})^2}{I}$$

### Least squares ﬁtting
另外需要注意的是，很多文獻都是以最小二乘法來討論擬合的。我們使用maximum
likelihood來擬和正態分佈的平均參數$$\mu$$。將前面式子將$$\sigma^2$$視為常數可以得到
$$\hat{\mu} = argmax_{\mu,\sigma^2} [-0.5Ilog[2\pi] - 0.5Ilog(\sigma^2) - 0.5\sum_{i=1}^I\frac{(x_i-\mu)^2}{\sigma^2}]$$

$$= argmax_{\mu,\sigma^2} [-\sum_{i=1}^I(x_i-\mu)^2]$$

$$= argmax_{\mu,\sigma^2} [\sum_{i=1}^I(x_i-\mu)^2]$$

也就是說least squares ﬁtting和使用maximum likelihood 估計常態分布的均值參數是等價的。


## log likehood好處
likelihood function可以看到是許多乘法的結果，因此微分很不好算。
由於log函數是monotonic的關係，所以likelihood function經過log轉換後的最大值會在相同的地方，而且經過log轉換後乘法變成加法，因此微分變得容易許多。

https://bookdown.org/dereksonderegger/571/13-maximum-likelihood-estimation.html#likelihood-function

https://math.stackexchange.com/questions/3053131/why-are-the-local-extrema-of-a-log-transformed-function-equal-to-local-extrema-o  

https://towardsdatascience.com/log-loss-function-math-explained-5b83cd8d9c83

### 方法二Maximum posteriori estimation
根據前面的定義，Maximum posteriori estimation的cost function為
$$\hat{\mu}, {\hat\sigma}^2 = argmax_{\mu,\sigma^2} [\prod_{i=1}^I Pr(x_i|\mu,\sigma^2)Pr(\mu,\sigma^2)]$$

$$= argmax_{\mu,\sigma^2} [\prod_{i=1}^I Norm_{x_i}[\mu,\sigma^2]NormInvGam_{\mu,\sigma^2}[\alpha,\beta,\gamma,\delta]]$$

在這裡我們選擇我們選擇了normal inverse gamma prior，其參數為α，β，γ，δ（圖4.4），因為它與normal distribution共軛。 

在式子中的prior如下
$$Pr(\mu,\sigma^2) = \frac{\sqrt{\gamma}}{\sigma\sqrt{2\pi}}\frac{\beta^{\alpha}}{\Gamma(\alpha)}(\frac{1}{\sigma^2})^{\alpha + 1}exp[-\frac{2\beta + \gamma(\delta-\mu)^2}{2\sigma^2}]$$

而posterior distribution 與likelihood和prior的乘積成正比（見圖4.5），在與數據一致且先驗可信的區域具有最高的機率密度。

而跟maximum likelihood一樣我們利用把式子取log來計算最大值。式子如下
$$\hat{\mu}, {\hat\sigma}^2 = argmax_{\mu,\sigma^2} [\sum_{i=1}^I log [Norm_{x_{i}}[\mu, {\sigma}^2]] + log [NormInvGam_{\mu,\sigma^2}[\alpha,\beta,\gamma,\delta]]]$$

要找到MAP(maximum a posteriori)我們將式子拆成兩段並且分別對$$\mu$$和$$\sigma^2$$做偏微分。式子如下
$$\hat{\mu} = \frac{\sum_{i=1}^I x_i + \gamma\delta}{I + \gamma}$$  

$$\hat{\sigma}^2 = \frac{\sum_{i=1}^I (x_i-\hat{\mu})^2 + 2\beta + \gamma(\delta-\hat{\mu})^2}{I + 3 + 2\alpha}$$

而平均數$\hat{\mu}$的解答可以進一步簡化
$$\hat{\mu} = \frac{I\bar{x} + \gamma\delta}{I + \gamma}$$

這式子是兩項的加權平均值，第一項是資料的平均$\bar{x}$並且以訓練樣本的數量$I$為權重，第二項是先驗分佈的參數$\delta$並且以先驗分佈的參數$\gamma$為權重

>這裡給我們一些MAP(maximum a posteriori)的洞察。
>1. 當資料數量越多時，MAP的解會越接近資料平均(也就是ML(Maximum likelihood)的解)
>2. 當資料數量少一些的時候，MAP的解會在ML和prior的中間
>3. 當完全沒有資料的時候，MAP的解就是proir

