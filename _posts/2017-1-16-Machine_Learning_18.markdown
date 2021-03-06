---
layout: post
title:  机器学习（十八）——推荐系统进阶
category: ML 
---

## Gibbs Sampling

这个算法虽然以Gibbs命名，但却是Geman兄弟于1984年研究Gibbs random field时，发现的算法。

>注：Josiah Willard Gibbs，1839~1903，美国科学家。他在物理、化学和数学方面都有重大理论贡献。耶鲁大学博士和教授。统计力学的创始人。

>Donald Jay Geman，1943年生，美国数学家。美国西北大学博士，布朗大学教授。随机森林算法的早期研究者之一。

>Stuart Alan Geman，1949年生，美国数学家。MIT博士，约翰霍普金斯大学教授。美国科学院院士。最早将Markov random field（MRF）应用于机器视觉和机器学习。

因为高维空间中，沿坐标轴方向上的两点之间的转移，满足细致平稳条件。因此，Gibbs Sampling的核心就是沿坐标轴循环迭代采样，该算法收敛之后的采样点即符合指定概率分布。

这里需要特别指出的是，Gibbs Sampling比Metropolis–Hastings算法高效的原因在于：Gibbs Sampling每次沿坐标轴的转移是必然会被接受的，即$$\alpha=1$$。

## Unigram Model

假设我们的词典中一共有V个词$$v_1,v_2,\cdots v_V$$，那么最简单的Unigram Model是定义一个V面的骰子，每抛一次骰子，抛出的面就对应产生一个词。

频率学派的Unigram Model如下图：

![](/images/article/unigram-model.jpg)

贝叶斯学派的Unigram Model如下图：

![](/images/article/dirichlet-multinomial-unigram.jpg)

这里使用Dirichlet分布的原因在于，数据采用多项分布，而Dirichlet分布正好是多项分布的共轭先验分布。

$$Dir(\overrightarrow{p}\mid \overrightarrow{\alpha})+MultCount(\overrightarrow{n})=Dir(\overrightarrow{p}\mid \overrightarrow{\alpha}+\overrightarrow{n})$$

和wiki上对Dirichlet分布的pdf函数的描述（参见《数学狂想曲（二）》中的公式1）不同，rickjin在这里采用了如下定义：

$$Dir(\overrightarrow{p}\mid \overrightarrow{\alpha})= 
\frac{1}{\Delta(\overrightarrow{\alpha})} \prod_{k=1}^V p_k^{\alpha_k -1}， 
\quad \overrightarrow{\alpha}=(\alpha_1, \cdots, \alpha_V)$$

其中：

$$\Delta(\overrightarrow{\alpha}) = 
\int \prod_{k=1}^V p_k^{\alpha_k -1} d\overrightarrow{p}$$

可以看出两种描述中的$$\mathrm{B}(\boldsymbol\alpha)$$和$$\Delta(\overrightarrow{\alpha})$$是等价的。

下面我们来证明这个结论，即：

$$\mathrm{B}(\boldsymbol\alpha)=\int \prod_{k=1}^V p_k^{\alpha_k -1} d\overrightarrow{p}\tag{1}$$

**证明**：这里为了简化问题，令V=3。则根据《数学狂想曲（二）》中的公式1可得：

$$Dir(\overrightarrow{p}\mid \overrightarrow{\alpha})= 
\frac{1}{\mathrm{B}(\alpha_1,\alpha_2,\alpha_3)}  p_1^{\alpha_1 -1}p_2^{\alpha_2 -1}p_3^{\alpha_3 -1}$$

对该pdf进行积分：

$$\iiint\frac{1}{\mathrm{B}(\alpha_1,\alpha_2,\alpha_3)}p_1^{\alpha_1 -1}p_2^{\alpha_2 -1}p_3^{\alpha_3 -1}\mathrm{d}p_1\mathrm{d}p_2\mathrm{d}p_3=\frac{1}{\mathrm{B}(\alpha_1,\alpha_2,\alpha_3)}\int p_1^{\alpha_1 -1}p_2^{\alpha_2 -1}p_3^{\alpha_3 -1}\mathrm{d}\overrightarrow{p}$$

由pdf的定义可知，上面的积分值为1。

因此：

$$\mathrm{B}(\alpha_1,\alpha_2,\alpha_3)=\int p_1^{\alpha_1 -1}p_2^{\alpha_2 -1}p_3^{\alpha_3 -1}\mathrm{d}\overrightarrow{p}$$

证毕。

从上面的证明过程，可以看出公式1并不是恒等式，而是在Dirichlet分布下才成立的等式。这也是共轭先验分布能够简化计算的地方。

## PLSA

Probabilistic Latent Semantic Analysis是Thomas Hofmann于1999年在UCB读博期间提出的算法。

原文：

https://dslpitt.org/uai/papers/99/p289-hofmann.pdf

示意图：

![](/images/article/plsa-doc-topic-word.jpg)

PLSA将生成文档的过程，分为两个步骤：

1.生成文档的主题。（doc->topic）

2.根据主题生成相关的词.（topic->word）

第m篇文档$$d_m$$中的每个词的生成概率为：

$$p(w\mid d_m) = \sum_{z=1}^K p(w\mid z)p(z\mid d_m) = \sum_{z=1}^K \varphi_{zw} \theta_{mz}$$

## LDA

利用贝叶斯学派的观点改造PLSA，可得：

![](/images/article/lda-dice.jpg)

LDA生成模型包含两个过程：

1.生成第m篇文档中的所有词对应的topics。

$$\overrightarrow{\alpha}\xrightarrow[Dirichlet]{} \overrightarrow{\theta}_m \xrightarrow[Multinomial]{} \overrightarrow{z}_{m}$$

2.K个由topics生成words的独立过程。

$$\overrightarrow{\beta} \xrightarrow[Dirichlet]{} \overrightarrow{\varphi}_k \xrightarrow[Multinomial]{} \overrightarrow{w}_{(k)}$$

因此，总共就是$$M+K$$个Dirichlet-Multinomial共轭结构。

LDA的Gibbs Sampling图示：

![](/images/article/gibbs-path-search.jpg)

LDA模型的目标有两个：

**训练模型**：估计模型中的参数：$$\overrightarrow{\varphi}_1, \cdots, \overrightarrow{\varphi}_K$$和$$\overrightarrow{\theta}_1, \cdots, \overrightarrow{\theta}_M$$。

由于参数$$\overrightarrow{\theta}_m$$是和训练语料中的每篇文档相关的，对于我们理解新的文档并无用处，所以工程上最终存储LDA模型时，一般没有必要保留。

这一步实际上是一个**聚类**的过程。

**使用模型**：对于新来的一篇文档$$doc_{new}$$，我们能够计算这篇文档的topic分布$$\overrightarrow{\theta}_{new}$$。

从最终给出的算法可以看出，虽然LDA用到了MCMC和Gibbs Sampling算法，但最终目的并不是生成符合相应分布的随机数，而是求出模型参数$$\overrightarrow{\varphi}$$的值，并用于预测。

这一步实际上是一个**分类**的过程。可见，LDA不仅可用于聚类，也可用于分类，是一种无监督的学习算法。

## 如何确定LDA的topic个数

这个问题上，业界最常用的指标包括Perplexity，MPI-score等。简单的说就是Perplexity越小，且topic个数越少越好。

从模型的角度解决主题个数的话，可以在LDA的基础上融入嵌套中餐馆过程(nested Chinese Restaurant Process)，印度自助餐厅过程(Indian Buffet Process)等。因此就诞生了这样一些主题模型：

1. hierarchical Latent Dirichlet Allocation (hLDA)  (2003_NIPS_Hierarchical topic models and the nested Chinese restaurant process)

2. hierarchical Dirichlet process (HDP)  (2006_NIPS_Hierarchical dirichlet processes)

3. Indian Buffet Process Compound Dirichlet Process (ICD)  (2010_ICML_The IBP compound Dirichlet process and its application to focused topic modeling)

4. Non-parametric Topic Over Time (npTOT)  (2013_SDM_A nonparametric mixture model for topic modeling over time)

5. collapsed Gibbs Samplingalgorithm for the Dirichlet Multinomial Mixture Model (GSDMM)  (2014_SIGKDD_A Dirichlet Multinomial Mixture Model-based Approach for Short Text Clustering)

这些主题模型都被叫做非参数主题模型(Non-parametric Topic Model)，最初可追溯到David M. Blei于2003年提出hLDA那篇文章(2003_NIPS_Hierarchical topic models and the nested Chinese restaurant process)。非参数主题模型是基于贝叶斯概率的与参数无关的主题模型。这里的参数无关主要是指模型本身可以“**随着观测数据的增长而相应调整**”，即主题模型的主题个数能够随着文档数目的变化而相应调整，无需事先人为指定。

参考：

https://www.zhihu.com/question/32286630

怎么确定LDA的topic个数？

http://blog.csdn.net/luo123n/article/details/48902815

Perplexity详解

## LDA漫游指南

除了rickjin的《LDA数学八卦》之外，马晨写的《LDA漫游指南》也是这方面的中文新作。

该书的数学推导部分主要沿用rickjin的内容，但加入了Blei提出的变分贝叶斯方法。此外，还对LDA的代码实现、并行计算和大数据处理进行了深入的讨论。

## 参考

http://www.arbylon.net/publications/text-est.pdf

《Parameter estimation for text analysis》，Gregor Heinrich著

http://www.inference.phy.cam.ac.uk/itprnn/book.pdf

《Information Theory, Inference, and Learning Algorithms》，David J.C. MacKay著

关于MCMC和Gibbs Sampling的更多的内容，可参考《Neural Networks and Learning Machines》，Simon Haykin著。该书有中文版。

>注：Sir David John Cameron MacKay，1967～2016，加州理工学院博士，导师John Hopfield，剑桥大学教授。英国能源与气候变化部首席科学顾问，英国皇家学会会员。在机器学习领域和可持续能源领域有重大贡献。

>Simon Haykin，英国伯明翰大学博士，加拿大McMaster University教授。初为雷达和信号处理专家。自适应信号处理领域的权威。80年代中后期，转而从事神经计算方面的工作。加拿大皇家学会会员。

http://www.cs.cmu.edu/~epxing/Class/10708-14/lectures/lecture17-MCMC.pdf

http://max.book118.com/html/2015/0513/16864294.shtm

基于LDA分析的词聚类算法

http://www.doc88.com/p-9159009103987.html

基于LDA的博客分类算法

http://blog.csdn.net/sinat_26917383/article/details/52095013

基于LDA的Topic Model变形+一些NLP开源项目

https://mp.weixin.qq.com/s/74lXwDg9H_dyOubfXVn2Bw

一文详解LDA主题模型

https://mp.weixin.qq.com/s/_bAiiJqPjPPMzJ0hiuCkOg

通过Python实现马尔科夫链蒙特卡罗方法的入门级应用

https://mp.weixin.qq.com/s/BJaRUnpcPe8iybSz14gabw

一文读懂如何用LSA、PSLA、LDA和lda2vec进行主题建模

# 推荐系统进阶

除了《机器学习（十三～十五）》提及的ALS和PCA之外，相关的算法还包括：

## FM：Factorization Machines

Factorization Machines是Steffen Rendle于2010年提出的算法。

>注：Steffen Rendle，弗赖堡大学博士，现为Google研究员。libFM的作者，被誉为推荐系统的新星。

FM算法实际上是一大类与矩阵分解有关的算法的广义模型。

参考文献1是Rendle本人的论文，其中有章节证明了SVD++、PITF、FPMC等算法，都是FM算法的特例。《机器学习（十四）》中提到的ALS算法，也是FM的特例。

参考文献2是国人写的中文说明，相对浅显一些。

参考：

https://www.ismll.uni-hildesheim.de/pub/pdfs/Rendle2010FM.pdf

http://blog.csdn.net/itplus/article/details/40534885

Factorization Machines 学习笔记（一）预测任务

https://tech.meituan.com/deep-understanding-of-ffm-principles-and-practices.html

深入FFM原理与实践

https://github.com/aksnzhy/xlearn

这是一个集成了FM和FFM等算法的库

## PITF

配对互动张量分解（Pairwise Interaction Tensor Factorization）算法，也是最早由Rendle引入推荐系统领域的。

论文：

http://www.wsdm-conference.org/2010/proceedings/docs/p81.pdf

## 其他

https://mp.weixin.qq.com/s/7yjA3_oCI5nSH4tv04BIhQ

HFT

https://mp.weixin.qq.com/s/gHKOArFzUM9Zn8hEsA-1wQ

FISM

https://mp.weixin.qq.com/s/VymwTuKq86JP2PL4v8LyyQ

POI by Friends

https://mp.weixin.qq.com/s/LnV-Oq3pCCeMk9RRhha-Aw

GLSLIM

https://mp.weixin.qq.com/s/xnJq-aBAZW22tP7RQylKLw

iCD

https://mp.weixin.qq.com/s/-IPwfrBz1dtYDupuGv4IjQ

Ensemble

https://mp.weixin.qq.com/s/SC8kNYvexetmDuxfQvwSDw

CKE

https://mp.weixin.qq.com/s/bu9rSno_WmHHisE3lzYnqg

ConvMF

