# CH09 EM算法及其推广

[TOC]

## 前言

### 章节目录

1. EM算法的引入
   1. EM算法
   1. EM算法的导出
   1. EM算法在非监督学习中的应用
1. EM算法的收敛性
1. EM算法在高斯混合模型学习中的应用
   1. 高斯混合模型
   1. 高斯混合模型参数估计的EM算法
1. EM算法的推广
   1. F函数的极大极大算法

### 导读

- EM算法可以用于**生成模型**的非监督学习, EM算法是个一般方法, 不具有具体模型.

  > EM算法是一种迭代算法, 用于含有隐变量的概率模型的极大似然估计,或极大后验概率估计. 

- 这里面注意体会不同变量的大小以及对应的取值范围.

- 一个$m\times n\times k$的矩阵可能可以划分成$n$个$m\times k$的形式, 这点理解下.

- 这部分推导有很多求和, 注意体会是按照**样本**做的, 还是按照**模型**做的

- 如果对PDF的概念不清楚, 高斯分布, 边缘概率分布, 协方差矩阵不清楚, 可以在这个章节从GMM的角度扩展阅读下, 一定会有收货.



## 符号说明

> 一般地, 用$Y$表示观测随机变量的数据, $Z$表示隐随机变量的数据. $Y$和$Z$一起称为**完全数据**(complete-data), 观测数据$Y$又称为**不完全数据**(incomplete-data)

上面这个概念很重要, Dempster在1977年提出EM算法的时候文章题目就是<Maximum likelihood from incomplete data vi the EM algorithm>, 具体看书中本章参考文献[1]

>假设给定观测数据$Y$, 其概率分布是$P(Y|\theta)$, 其中$\theta$是需要估计的模型参数
>那么不完全数据$Y$的似然函数是$P(Y|\theta)$, 对数似然函数是$L(\theta)=\log P(Y|\theta)$
>
>假设$Y$和$Z$的联合概率分布是$P(Y,Z|\theta)$, 那么完全数据的对数似然函数是$\log P(Y,Z|\theta)$

上面这部分简单对应一下

|                   |                                       |                       |                                  |
| ----------------- | ------------------------------------- | --------------------- | -------------------------------- |
|                   | 观测数据$Y$                           | 不完全数据$Y$         |                                  |
| 不完全数据$Y$     | 概率分布$P(Y|\theta)$                 | 似然函数$P(Y|\theta)$ | 对数似然函数$ \log P(Y|\theta)$  |
| 完全数据 $(Y, Z)$ | $Y$和$Z$的联合概率分布$P(Y,Z|\theta)$ | 似然函数$P(Y,Z|\theta)$ | 对数似然函数$\log P(Y,Z|\theta)$ |

观测数据$Y$

有一点要注意下, 这里没有出现$X$, 在**9.1.3**节中有提到一种理解

> - 有时训练数据只有输入没有对应的输出${(x_1,\cdot),(x_2,\cdot),\dots,(x_N,\cdot)}$, 从这样的数据学习模型称为非监督学习问题.
> - EM算法可以用于生成模型的非监督学习.
> - 生成模型由联合概率分布$P(X,Y)$表示, 可以认为非监督学习训练数据是联合概率分布产生的数据. $X$为观测数据, $Y$为未观测数据.

有时候, 只观测显变量看不到关系, 就需要把隐变量引进来.

## 问题描述

书中用例子来介绍EM算法的问题, 并给出了EM算法迭代求解的过程, 具体例子描述见**例9.1**,解的过程记录在这里. 

三硬币模型可以写作
$$
\begin{equation}
\begin{aligned}
P(y|\theta)&=\sum_z P(y,z|\theta) \\
&=\sum_z P(z|\theta)P(y|z,\theta) \\
&=\pi p^y (1-p)^{1-y} + (1-\pi)q^y(1-q)^{1-y}
\end{aligned}
\end{equation}
$$
以上

1. 随机变量$y$是观测变量, 表示一次试验观测的结果是1或0
1. 随机变量$z$是隐变量, 表示未观测到的掷硬币$A$的结果
1. $\theta=(\pi,p,q)$是模型参数
1. 这个模型是**以上数据**(1,1,0,1,0,0,1,0,1,1)的生成模型.



观测数据表示为$Y=(Y_1, Y_2, Y_3, \dots, Y_n)^T$, 未观测数据表示为$Z=(Z_1,Z_2, Z_3,\dots, Z_n)^T$, 则观测数据的似然函数为
$$
P(Y|\theta) = \sum\limits_{Z}P(Z|\theta)P(Y|Z,\theta)
$$
即
$$
P(Y|\theta)=\prod\limits^{n}_{j=1}[\pi p^{y_j}(1-p)^{1-y_j}+(1-\pi)q^{y_j}(1-q)^{1-y_j}]
$$
考虑求模型参数$\theta=(\pi,p,q)$的极大似然估计, 即
$$
\hat \theta = \arg\max\limits_{\theta}\log P(Y|\theta)
$$


## EM算法的解释

注意这里EM不是模型, 是个一般方法, 不具有具体的模型.

### PRML

$kmeans \rightarrow GMM \rightarrow EM$

所以, EM应用举例子为kmeans也OK.

### 统计学习方法

1. $MLE \rightarrow B$
1. $F$函数的极大-极大算法

## 高斯混合模型

**混合模型**, 有多种, 高斯混合模型是最常用的. 

高斯混合模型(Gaussian Mixture Model)是具有如下**概率分布**的模型:
$$
P(y|\theta)=\sum\limits^{K}_{k=1}\alpha_k\phi(y|\theta_k)
$$
其中, $\alpha_k$是系数, $\alpha_k\ge0$, $\sum\limits^{K}_{k=1}\alpha_k=1$, $\phi(y|\theta_k)$是**高斯分布密度**, $\theta_k=(\mu,\sigma^2)$
$$
\phi(y|\theta_k)=\frac{1}{\sqrt{2\pi}\sigma}\exp\left(-\frac{(y-\mu_k)^2}{2\sigma_k^2}\right)
$$
上式表示第k个**分**模型.

以上, 注意几点:
1. GMM的描述是概率分布, 形式上可以看成是加权求和

1. 加权求和的权重$\alpha$满足$\sum_{k=1}^K\alpha_k=1$的约束

1. 求和符号中除去权重的部分, 是高斯分布密度(PDF). 高斯混合模型是一种$\sum(权重\times 分布密度)=分布$的表达
  高斯混合模型的参数估计是EM算法的一个重要应用, 隐马尔科夫模型的非监督学习也是EM算法的一个重要应用. 

1. 书中描述的是一维的高斯混合模型, d维的形式如下[^2]:
  $$
  \phi(y|\theta_k)=\frac{1}{\sqrt{(2\pi)^d|\Sigma|}}\exp\left(-\frac{(y-\mu_k)^T\Sigma^{-1}(y-\mu_k)}{2}\right)
  $$


### GMM的EM算法

问题描述:

已知观测数据$y_1, y_2, \dots, y_N$, 由高斯混合模型生成
$$
P(y|\theta)=\sum_{k=1}^K\alpha_k\phi(y|\theta_k)
$$
其中, $\theta=(\alpha_1,\alpha_2,\dots,\alpha_K;\theta_1,\theta_2,\dots,\theta_K)$

使用EM算法估计GMM的参数$\theta$

#### 1. 明确隐变量

- 观测数据$y_j, j=1,2,\dots,N$这样产生, 是**已知的**:
  首先依概率$\alpha_k$选择第$k$个高斯分布分模型$\phi(y|\theta_k)$;
  然后依第$k$个分模型的概率分布$\phi(y|\theta_k)$生成观测数据$y_j$

- 反映观测数据$y_j$来自第$k$个分模型的数据是**未知的**,$k=1,2,\dots,K$以**隐变量$\gamma_{jk}$**表示
  **注意这里$\gamma_{jk}$的维度**
  $$
  \gamma_{jk}=
  \begin{cases}
  1, &第j个观测来自第k个分模型\\
  0, &否则
  \end{cases}\\
  j=1,2,\dots,N; k=1,2,\dots,K; \gamma_{jk}\in\{0,1\}
  $$

- 完全数据为$(y_j,\gamma_{j1},\gamma_{j2},\dots,\gamma_{jK},k=1,2,\dots,N)$

- 完全数据似然函数
  $$
  \begin{aligned}
  P(y,\gamma|\theta)=&\prod_{j=1}^NP(y_j,\gamma_{j1},\gamma_{j2},\dots,\gamma_{jK}|\theta)\\
  =&\prod_{k=1}^K\prod_{j=1}^N\left[\alpha_k\phi(y_j|\theta_k)\right]^{\gamma_{jk}}\\
  =&\prod_{k=1}^K\alpha_k^{n_k}\prod_{j=1}^N\left[\phi(y_j|\theta_k)\right]^{\gamma_{jk}}\\
  =&\prod_{k=1}^K\alpha_k^{n_k}\prod_{j=1}^N\left[\frac{1}{\sqrt{2\pi}\sigma_k}\exp\left(-\frac{(y_j-\mu_k)^2}{2\sigma^2}\right)\right]^{\gamma_{jk}}\\
  \end{aligned}
  $$
  其中$n_k=\sum_{j=1}^N\gamma_{jk}, \sum_{k=1}^Kn_k=N$

- 完全数据对数似然函数
  $$
  \log P(y,\gamma|\theta)=\sum_{k=1}^K\left\{n_k\log \alpha_k+\sum_{j=1}^N\gamma_{jk}\left[\log \left(\frac{1}{\sqrt{2\pi}}\right)-\log \sigma_k -\frac{1}{2\sigma^2}(y_j-\mu_k)^2\right]\right\}
  $$




#### 2. E步,确定Q函数

把Q函数表示成参数形式
$$
\begin{aligned}
Q(\theta,\theta^{(i)})=&E[\log P(y,\gamma|\theta)|y,\theta^{(i)}]\\
=&\color{green}E\color{black}\left\{\sum_{k=1}^K\left\{\color{red}n_k\color{black}\log \alpha_k+\color{blue}\sum_{j=1}^N\gamma _{jk}\color{black}\left[\log \left(\frac{1}{\sqrt{2\pi}}\right)-\log \sigma _k-\frac{1}{2\sigma^2(y_j-\mu_k)^2}\right]\right\}\right\}\\
=&\sum_{k=1}^K\left\{\color{red}\sum_{j=1}^{N}(\color{green}E\color{red}\gamma_{jk})\color{black}\log \alpha_k+\color{blue}\sum_{j=1}^N(\color{green}E\color{blue}\gamma _{jk})\color{black}\left[\log \left(\frac{1}{\sqrt{2\pi}}\right)-\log \sigma _k-\frac{1}{2\sigma^2(y_j-\mu_k)^2}\right]\right\}\\
\end{aligned}
$$

$$
\begin{align}
\hat \gamma _{jk}= &E(\gamma_{jk}|y,\theta)=P(\gamma_{jk}=1|y,\theta)\\
=&\frac{P(\gamma_{jk}=1,y_j|\theta)}{\sum_{k=1}^KP(\gamma_{jk}=1,y_j|\theta)}\\
=&\frac{P(y_j|\gamma_{jk}=1,\theta)P(\gamma_{jk}=1|\theta)}{\sum_{k=1}^KP(y_j|\gamma_{jk}=1,\theta)P(\gamma_{jk}=1|\theta)}\\
=&\frac{\alpha_k\phi(y_j|\theta_k)}{\sum_{k=1}^K\alpha_k\phi(y_j|\theta_k)}
\end{align}
$$



​    这部分内容就是搬运了书上的公式, 有几点说明:

1. 注意这里$E(\gamma_{jk}|y,\theta)$,记为$\hat\gamma_{jk}$,E步求的**期望**就是这个.

1. 对应理解一下上面公式中的红色,蓝色和绿色部分.

1. 这里用到了$n_k=\sum_{j=1}^N\gamma_{jk}$

1. $\hat \gamma_{jk}$为分模型$k$对观测数据$y_j$的响应度. 这里, 第一行参考伯努利分布的期望.

$$
Q(\theta,\theta^{(i)})=\sum_{k=1}^Kn_k\log \alpha_k+\sum_{j=1}^N\hat \gamma_{jk}\left[\log \left(\frac{1}{\sqrt{2\pi}}\right)-\log \sigma_k-\frac{1}{2\sigma_k^2}(y_j-\mu_k)^2\right]
$$

其中$i$表示第$i$步迭代

其实抄公式没有什么意义,主要是能放慢看公式的速度. 和图表一样, 公式简洁的表达了很多含义, 公式中也许更能体会到数学之美.

#### 3. M步

求函数$Q(\theta,\theta^{(i)})$对$\theta$的极大值, 分别求$\sigma, \mu, \alpha$
$$
\theta^{(i+1)}=\arg\max_\theta Q(\theta,\theta^{(i)})
$$

- $Q$对$\mu_k, \sigma^2$求偏导得到$\hat\mu_k, \hat \sigma_k^2$
- $\sum_{k=1}^K\alpha_k=1$条件下求偏导得到$\alpha_k$

TODO: 推导

#### 4. 停止条件

重复以上计算, 直到对数似然函数值不再有明显的变化为止.

### 算法9.2

这部分摘要总结了前面的公式.

因为公式比较集中, 方便对比, 注意体会以下两个方面:

1. 这几个公式中待求的变量的维度和角标的关系.
1. 这里面有求和, 前面提到过, 注意体会每一步刷的是模型, 还是样本

### Kmeans

另外, 直觉上看, GMM最直观的想法就是Kmeans, 那么:

1. 在Kmeans常见的描述中都有距离的概念, 对应在算法9.2 的描述中, 该如何理解?
   这里面距离对应了方差
1. 那么又是怎么在每轮刷过距离之后, 重新划分样本的分类呢?
   这里对应了响应度, 响应度对应了一个$j \times k$的矩阵, 记录了每一个$y_j$ 对第$k$个模型的响应度, 可以理解为划分了类别.

### K怎么定?

- 手肘法
- Gap Statistics[^1]

## 广义期望极大

广义期望极大(generalized expectation maximization, $GEM​$)

## 其他

1. 关于习题9.3 
   GMM模型的参数($\alpha _k, \mu _k, \sigma^2_k $)应该是$3k$个, 题目9.3中提出两个分量的高斯混合模型的5个参数, 是因为参数$\alpha_k$满足$\sum_{k=1}^K\alpha _k=1$ 
1. 

## 参考

1. [EM Algorithm](https://en.wikipedia.org/wiki/Expectation%E2%80%93maximization_algorithm)

2. [Sklearn Gaussian Mixed Model](http://scikit-learn.org/stable/modules/mixture.html)

3. [^1]: [Gap Statistics](https://web.stanford.edu/~hastie/Papers/gap.pdf)

4. [^2]: [多元正态分布](https://zh.wikipedia.org/wiki/%E5%A4%9A%E5%85%83%E6%AD%A3%E6%80%81%E5%88%86%E5%B8%83)
