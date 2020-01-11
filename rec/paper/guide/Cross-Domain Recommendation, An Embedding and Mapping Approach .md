## Cross-Domain Recommendation：An Embedding and Mapping Approach

IJCAI-17

------------------

### Abstract

跨域推荐是解决推荐领域**数据稀疏性**（sparsity）的一种方法

leveraging feedbacks or ratings from multiple domains to improve recommendation performance in a collective manner

**propose** ：an Embedding and Mapping framework for Cross-Domain Recommendation, called EMCDR.

**distinguishes** ：用多层感知机捕获域间的非线性映射函数

​							保证在单域中数据数据稀疏产生噪声对模型造成的影响（robustness）。

**demonstrate** ：在很多 two corss-domain 场景下 EMCDR 要比 state-of-the-art（当前最高水平） 性能要好



### 1 Introduction

- **已存在的跨域推荐的类型：**

  - 非对称：利用辅助领域知识降低目标领域知识的稀疏性。辅助领域充当先验或正则化项作用，关键点在于哪些 knowledge 可以被迁移到目标领域。
  - 对称：将两领域同等看待，互为补充。

  > https://blog.csdn.net/qq_32813967/article/details/81087973
  >
  > 笔者认为，两种分类的区别在于：
  >
  > 非对称方式中源域的知识足够多而目标域知识相对少，源域需要的知识目标域无法提供，而目标域需要源域的知识。
  >
  > 而对称方式中源域和目标域知识都不太多，有一定交集，也有各自的部分，需要互为补充提高精度。

  作者研究的是**对称性跨域**



**作者研究的 key issues**：

- How to represent the mapping function across domains, in linear or nonlinear format? 
- Which part of data should be leveraged to learn the mapping function, all available data in both domains or just part of them?

**训练映射函数容易遇到的问题**：

- MLP容易过拟合
- 不活跃的user和item对训练模型有害



### 2 The Proposed EMCDR Framework

User $U = \{u_1,u_2,...\}$

Item $V = \{v_1,v_2,...\}$

$R^s$ and $R^t$ be two rating matrices from the source and target domains respectively

$R^s_{ij}$  is the rating that user $u_i$ gives to item $v_j$ in the source domain  , and $R^t_{ij}$  is the corresponding rating in the target domain

EMCDR 模型流程如下：

<img src="C:/Users/lenovo-aa/Desktop/推荐系统/study/论文阅读/推荐系统/img/image-cro.png" alt="image-20200106122107154" style="zoom:50%;" />

> 笔者认为
>
> 第一步：找出 source 域 和 target 域 的latent factor of users and items
>
> 第二步：找出 source 域 和 target 域 的映射函数
>
> 第三步：使用 函数 对 target 域 的 user and item 进行推荐

### 3 The EMCDR Models

implement the proposed EMCDR framework into different models (MF & BPR) with two different latent factor models, and two latent space mapping functions.

#### 3.1 Latent Factor Modeling（Embedding）

**MF：rating-oriented**

R： $|u| \times |v|$ rating matrix

$R_{ij}$ ：user-item pair

K：潜在因子的维度

U：$K \times |u|$   user潜在因子矩阵 ，其中第 i 列 $U_i$ 代表 用户 $u_i$ 的潜在因子

V：$K \times|v|$   item潜在因子矩阵 ，其中第 i 列 $V_i$ 代表 item $v_i$ 的潜在因子

$p\left(R_{i j} | U_{i}, V_{j} ; \sigma^{2}\right)=\mathcal{N}\left(R_{i j} | U_{i}^{T} V_{j}, \sigma^{2}\right)$

$\min _{U, V}\left(\sum_{i} \sum_{j}\left\|I_{i j} \cdot\left(R_{i j}-U_{i}^{T} V_{j}\right)\right\|_{F}^{2}\right.\left.+\lambda_{U} \sum_{i}\left\|U_{i}\right\|_{F}^{2}+\lambda_{V} \sum_{j}\left\|V_{j}\right\|_{F}^{2}\right)$

$U_{i} \leftarrow U_{i}-\eta \cdot\left\{\left(R_{i j}-U_{i}^{T} V_{j}\right) V_{j}+\lambda_{U} U_{i}\right\}$
$V_{j} \leftarrow V_{j}-\eta \cdot\left\{\left(R_{i j}-U_{i}^{T} V_{j}\right) U_{i}+\lambda_{V} V_{j}\right\}$



**BPR：ranking-oriented**

training set D：$ u\times v \times v$ composed by preference pairs ,from the original rating matix



$p\left(R_{i j}>R_{i l}\right)=\sigma\left(U_{i}^{T} V_{j}-U_{i}^{T} V_{l}\right)$

$\begin{aligned} \min _{U, V} &\left(\sum_{\left(u_{i}, v_{j}, v_{l}\right) \in D}-\ln \sigma\left(U_{i}^{T} V_{j}-U_{i}^{T} V_{l}\right)\right.\left.+\lambda_{U} \sum_{i}\left\|U_{i}\right\|_{F}^{2}+\lambda_{V} \sum_{j}\left\|V_{j}\right\|_{F}^{2}\right) \end{aligned}$

$\Theta \leftarrow \Theta+\eta \cdot\left(\frac{e^{-\hat{x}_{i j l}}}{1+e^{-\hat{x}_{i j l}}} \cdot \frac{\partial}{\partial \Theta} \hat{x}_{i j l}-\lambda_{\Theta} \Theta\right)$



------

MF 和 BPR 的参数可以使用随机梯度优化以上两个函数来进行估计 

从 模型中学到的潜在因子可以看做在潜在空间中的坐标

之后，利用这些坐标作为隐含特征去学习映射函数





#### 3.2 Latent Space Mapping

从 model 中，我们可以得到 潜在因子 $\{U_s,V_s,U_t,V_t\}$

<损失函数>

**Linear Mapping**

将映射函数定义为一个转置矩阵 M

使得 $M \times U_i^s$ 逼近 $U_i^t$ 

<M公式>



**MLP-based Nonlinear Mapping**

<公式>

用反向传播来训练参数的梯度



#### 3.3 Cross-domain Recommendation



### 4 Experiments

#### 4.1 Experiments Setup

**Datasets** ： MovieLens-Netflix(item-shared) & Douban(user-shared)

**Expriment Setup**：

- 随机删除目标域中一部分实体的所有 rating 信息，将它们作为跨域冷启动实体进行推荐。

- set different fractions for cold-start entities namely,10%,20%,30%,40%,and 50%

- 随机取样 L（10） 次，取其平均结果和标准差
- Dimension K set as 20，50，and 100
- 5-fold 交叉验证

**线性模型：**

- 对于 permutation matrix is $K \times K$
- regularization coefficient $\lambda_M = 0.01$ 

**非线性模型：**

- 一个隐含层，节点数为 $2 \times K $
- 输入输出的维度为 K
- 权重和偏置初始化根据 [Glorot and Bengio,2010]
- minibatch = 16
- 激活函数使用 tan-sigmoid function



**四种模型**

**与 CMF CST LFM 性能对比**



#### 4.2 Experimental Results

**Recommendation Performance**



**Mapping Function Learning**

> 用户的活跃度对mapping函数训练的影响

