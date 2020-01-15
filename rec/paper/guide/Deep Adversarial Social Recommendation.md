## 导读





## Abstract

Propose ：DASO

用户在 item域 和 social域表现是不一样的



## 1 Introduction

通常的社交推荐的方法：

- trust propagation
- 临近user聚集
- user-item、user-user 两个域使用一个公共的user表征

> 用户在 item域 和 social域表现是不一样的，所以尽管有user在两个域之间做连接，还是存在一定的问题



**challenge**：

- 两个域在进行 information 传播的同时，分别学习两个 user 表征

- 数据稀疏

  > （不太理解）作者说：在一开始的训练过程中，负样本还比较有效，但是在后面的过程中，负样本就不太能提供information去提高用户的表征。
  >
  > 我个人理解还是因为样本不平衡，负样本太少的原因，感觉推荐天生的特征就是样本不均衡，稀疏。
  >
  > 所以呢还是需要用GAN来拟合分布，并产生一定量的高级（or difficult）fake来作为real样本进行使用



> GAN ：shown great success across various domains due  to their ability to learn an underlying data distribution and generate synthetic samples 



**使用 social 域来增强 item 域的 user-item 表征**



## 2 The Proposed Framework

**这个理解很重要**

1 不代表 user 非常喜欢该 item，可能有一些其他的噪声才产生了一些交互

0 不代表 user 不喜欢该 item ，可能是 user 并没有发现该 item

> 使用GAN增强 interaction matrix R 和 social network S，为了预测一个没有出现过的 $r_{i,j} = 0$ in R ，可以理解为增强R和S之后，就可以用两个矩阵来进行评分，即判别 $u_i$ 是否会对 $v_j$ 产生兴趣



### 2.1 An Overview of the Proposed Framework

**Three components**：

- cyclic user modeling：**model user representations on two domains**
- item domain adversarial learning：learning of user and item representations
  - generator：给定 user 表征 ，生成(推荐、pick) items ，输出 fake pairs
- social domain adversarial learning：learning of user and user representations



### 2.2 Cyclic User Modeling

使用  MLP 转移 Social Information 到 Item Domain

$$P_i^{SI} = h^{S->I}\left(P_i^{S} \right) =\mathbf{W}_{L} \cdot\left(\cdots \cdot a\left(\mathbf{W}_{2} \cdot a\left(\mathbf{W}_{1} \cdot\mathbf{p}_{i}^{S}+\mathbf{b}_{1}\right)+\mathbf{b}_{2}\right) \ldots\right)+\mathbf{b}_{L}$$

> a ：activation function

**Bidirectional Mapping with Cycle Reconstuction**

双向映射导推：

- $$P_i^{I} —> h^{I->S}\left(P_i^{I} \right)—>h^{S->I}\left( h^{I->S}\left(P_i^{I} \right) \right) \approx P_i^{I}$$

- $$P_i^{S} —> h^{S->I}\left(P_i^{I} \right)—>h^{I->S}\left( h^{S->I}\left(P_i^{I} \right) \right) \approx P_i^{S}$$

loss function：
$$
\begin{aligned}
\mathcal{L}_{c y c}\left(h^{S \rightarrow I}, h^{I \rightarrow S}\right) &=\sum_{i=1}^{N}\left(\left\|h^{S \rightarrow I}\left(h^{I \rightarrow S}\left(\mathbf{p}_{i}^{I}\right)\right)-\mathbf{p}_{i}^{I}\right\|_{2}+\left\|h^{I \rightarrow S}\left(h^{S \rightarrow I}\left(\mathbf{p}_{i}^{S}\right)\right)-\mathbf{p}_{i}^{S}\right\|_{2}\right)
\end{aligned}
$$





### 2.3 Item Domain Adversarial Learning


$$
\begin{aligned}
& min _{\theta_{G}^{I}} max _{\phi_{D}^{I}} \mathcal{L}_{a d v}^{I}\left(G^{I}, D^{I}\right)\\
&\begin{array}{l}
{=\sum_{i=1}^{N}\left(\mathbb{E}_{v \sim p_{\text {real}}^{I}\left(\cdot | u_{i}\right)}\left[\log D^{I}\left(u_{i}, v ; \phi_{D}^{I}\right)\right]\right.} \\
{\left.+\mathbb{E}_{v \sim G^{I}\left(\cdot | u_{i} ; \theta_{G}^{I}\right)}\left[\log \left(1-D^{I}\left(u_{i}, v ; \phi_{D}^{I}\right)\right)\right]\right)}
\end{array}
\end{aligned}
$$

> 训练D
>
> - 如果是真正的 user-item paris，则希望分数越高
>
> - 如说是生成器生成的 user-item paris，则希望分数越低，那么第二项就会越接近（log1）,则式子整体会越大。
>
> 训练G
>
> - （只有第二项），最大化 D，则最小化整个式子
>
> 
>
> **期望 E** 暂时理解成 每次训练的时候一批一批地输入。 

**Discriminator**：区分 real user-item paris 和 生成的 user-item pairs 

以下是一个**sigmoid function**
$$
D^{I}\left(u_{i}, v_{j} ; \phi_{D}^{I}\right)=\sigma\left(f_{\phi_{D}^{I}}^{I}\left(\mathbf{x}_{i}^{I}, \mathbf{y}_{j}^{I}\right)\right)=
\frac{1}{1+\exp \left(-f_{\phi_{D}^{I}}^{I}\left(\mathbf{x}_{i}^{I}, \mathbf{y}_{j}^{I}\right)\right)}
$$

> $f_{\phi_{D}}^{I}$ is a score function $f_{\phi_{D}^{I}}^{I}\left(\mathbf{x}_{i}^{I}, \mathbf{y}_{j}^{I}\right)=\left(\mathbf{x}_{i}^{I}\right)^{T} \mathbf{y}_{j}^{I}+\mathbf{a}_{j}$

**用随机梯度下降进行优化**



**Generator**：尽可能拟合  $p_{\text {real}}^{I}\left(v | u_{i}\right)$ 分布，并根据 user 生成（挑选）出与其最相关的 items

以下是一个**softmax function**
$$
G^{I}\left(v_{j} | u_{i} ; \theta_{G}^{I}\right)=\frac{\exp \left(g_{\theta_{C}^{I}}^{I}\left(\mathbf{p}_{i}^{S I}, \mathbf{q}_{j}^{I}\right)\right)}{\sum_{v_{j} \in \mathcal{V}} \exp \left(g_{\theta_{C}^{I}}^{I}\left(\mathbf{p}_{i}^{S I}, \mathbf{q}_{j}^{I}\right)\right)}
$$

> $g_{\theta_{C}^{I}}^{I}$ is a score function reflecting the chance of $v_j$ being clicked/purchased by $u_i$
>
> $g_{\theta_{G}^{I}}^{I}\left(\mathbf{p}_{i}^{S I}, \mathbf{q}_{j}^{I}\right)=\left(\mathbf{p}_{i}^{S I}\right)^{T} \mathbf{q}_{j}^{I}+b_{j}$

作者注意到：给定一个user生成相关的item是离散的，所以不能使用随机梯度下降法来优化 $G_I$ ,所以采用了 **the policy gradient method** 

训练生成器 就是 最小化 ： $ \min _{\theta_{G}^{I}} \sum_{i=1}^{N}\left(\mathbb{E}_{v \sim G^{I}\left(\cdot | u_{i} ; \theta_{G}^{I}\right)}\left[\log \left(1-D^{I}\left(u_{i}, v ; \phi_{D}^{I}\right)\right)\right]\right)$

等价于最大化（误导判别器）：$\max _{\theta_{G}^{I}} \sum_{i=1}^{N}\left(\mathbb{E}_{v \sim G^{I}\left(\cdot | u_{i} ; \theta_{G}^{I}\right)}\left[\log \left(1+\exp \left(f_{\phi_{D}^{I}}^{I}\left(\mathbf{x}_{i}^{I}, \mathbf{y}_{j}^{I}\right)\right)\right)\right]\right)$



![image-20200112174545971](C:\Users\lenovo-aa\Desktop\第二周论文阅读\img\image-20200112174545971.png)



### 2.4 Social Domain Adversarial Learning

> 同 Item Domain 

### 2.5 The Objective Function

$$
\begin{aligned}
& min_{G^{I}, G^{S}, h^{S \rightarrow I}, h^{I \rightarrow S}}  max_{D^{I}, D^{S }}\mathcal{L}\\
&=F\left(G^{I}, D^{I}, G^{S}, D^{S}, h^{S \rightarrow I}, h^{I \rightarrow S}\right)\\
&=\mathcal{L}_{a d v}^{I}\left(G^{I}, D^{I}\right)+\mathcal{L}_{a d v}^{S}\left(G^{S}, D^{S}\right)+\lambda \mathcal{L}_{c y c}\left(h^{S \rightarrow I}, h^{I \rightarrow S}\right)
\end{aligned}
$$

> $\lambda$ 控制 cycle 策略的重要性和mapping function 的影响
>

使用 **RMSprop** 优化目标函数 

> 训练过程（e.g.）：固定 $D^I,G^S,D^S$ ，训练 $G^I $


$$
\begin{aligned}
\mathcal{L}_{BiGAN}=\mathcal{L}_{a d v}^{f}\left(G^{f}, D^{f}\right)+\mathcal{L}^{b}_{a d v}\left(G^{b}, D^{b}\right)+\lambda \mathcal{L}_{sim}\left(G^{f},G^{b}\right)
\end{aligned}
$$




## 3 Experiments

### 3.1 Experimental Settings

**Dataset**：Ciao and Epinions

把用户评分改为用户是否有feedback，如果有，则值为 1

80% train 10% validation 10% test

metrics ：Precision@K  &  NDCG@K   （越高性能越好）

size of representation d ：{8,16,32,64,128,256}

### 3.2 Performance Comparison of Recommender Systems

![image-20200112203736437](C:\Users\lenovo-aa\Desktop\第二周论文阅读\img\image-20200112203736437.png)

<img src="C:\Users\lenovo-aa\Desktop\第二周论文阅读\img\image-20200112203811675.png" alt="image-20200112203828033" style="zoom:50%;" />

<img src="C:\Users\lenovo-aa\Desktop\第二周论文阅读\img\image-20200112204027013.png" alt="image-20200112204027013" style="zoom:50%;" />

## 4 Related Work







## 5 Conclusion and Future Work

在训练 Generator 的时候，效率比较低下，考虑使用 hierarchical softmax 代替 softmax





## Qs

- GAN
- Precision@K  &  NDCG@K
- 激活函数种类
- 两数据集
- 梯度下降和反向传播



## OutLook

- cycle 部分直接用 GAN

- cycle 部分先用embedding或auto-encoder技术，再用MLP或GAN

- 替换S-I为S-T（两个数据集）

- 把GAN 换为双向GAN ，中间还是用MLP

  