# Q1:奇异值分解SVD的原理是怎样的，都有哪些应用场景？

## A1:

### 奇异值分解SVD的原理:

将不对称的矩阵A通过 $AA^{T}$ 和 $A^{T}A$ 得到两个对称的矩阵，对这2个对称矩阵分别求特征值和特征向量，两个对称矩阵的特征值相同，$AA^{T}$对应的特征向量称为左奇异矩阵P， $A^{T}A$对应的特征向量称为右奇异矩阵Q，特征值构成对角线矩阵$\Lambda $， $ P\Lambda Q^{T} = A$ 完成奇异值分解。

### 奇异值分解SVD应用场景:

+ 在推荐系统中，把高阶的评分矩阵通过SVD分解，用低秩的矩阵来逼近原来的评分矩阵，逼近的目标就是让预测的矩阵和原来的矩阵之间的误差平方最小。左奇异矩阵为User矩阵；右奇异矩阵为Item矩阵，利用由此得到的评分矩阵，进行推荐。
+ 奇异值分解能够简约数据，去除噪声和冗余数据。因此也常在机器学习中被用作降维方法。比如：在图像压缩中，可以使用奇异值分解使用低阶矩阵保留足够多的图像信息；

# Q2:funkSVD, BiasSVD，SVD++算法之间的区别是怎样的？

## A2：

三种方法都是对传统SVD的优化方法。

1. 优化的内容不同

   + 传统SVD分解要求矩阵是稠密的，而FunkSVD可以避开稀疏问题，**只关注与原来矩阵中有值的位置进行对比，不对所有元素进行对比**；

   + BiasSVD在FunkSVD的基础上，**增加了对用户偏好、商品偏好的考虑**，将与个性化无关的部分，设置为偏好(Bias)部分。从而解决以下3种问题：

     + 人员问题：参与打分的人员，有的可能比较好说话，打分都偏高；有的可能要求比较严格，打分都偏低；有的可能是水军等等，BiasSVD算法降低了这部分不能反映真实大众的想法的误差；
     + 作品问题：质量好的商品，打分会偏高；也反映不出真是大众的水平，BiasSVD算法也降低了这部分误差；
     + 全局问题：以电影为例，赶上某一个时间段内大众审美

   + SVD++ 在BiasSVD算法基础上进行了改进，增加**考虑用户的隐式反馈**。隐式反馈指虽然没有具体评分，但用户可能有点击、浏览等行为；

     

2. 目标函数不同

   因优化内容的不同 ，目标函数也不同：

   + FunkSVD目标是让用矩阵乘积得到的评分和用户的实际评分残差尽可能的小，使用均方差作为损失函数，同时为了防止过拟合，加入了L2正则化项，来寻找最终的矩阵P、Q。

   $$
   arg \underset{p_{i}q_{j}}{min} \sum _{i,j\varepsilon K}(m_{ij}-p_{i}^Tq_{j})^2+\lambda (\left \| p_{i} \right \|_{2}^2 + \left \| q_{j} \right \|_{2}^2)
   $$

   + BiasSVD 增加了对用户偏好bi（自身属性，与商品无关），商品偏好bj（自身属性，与用户无关）的正则项。

   $$
   arg \underset{p_{i}q_{j}}{min} \sum _{i,j\varepsilon K}(m_{ij}-\mu - b_{i} - b_{j} - p_{i}^{T}q_{j})^2 + \lambda (\left \| p_{i} \right \|_{2}^2 + \left \| q_{j} \right \|_{2}^2 + \left \| b_{i} \right \|_{2}^2) + \left \| b_{j} \right \|_{2}^2)
   $$

   + SVD++ 在BiasSVD算法基础上进行了改进，增加用户i所有的隐式反馈修正值之和：$\sum_{s\varepsilon N(i)} c_{sj}$

   $$
   arg \underset{p_{i}q_{j}}{min} \sum _{i,j\varepsilon K}(m_{ij}-\mu - b_{i} - b_{j} - p_{i}^{T}q_{j} - q_{j}^{T}|I(i)|^{-\frac{1}{2}}\sum_{s\varepsilon I(i)} y_s)^2 + \lambda (\left \| p_{i} \right \|_{2}^2 + \left \| q_{j} \right \|_{2}^2 + \left \| b_{i} \right \|_{2}^2) + \left \| b_{j} \right \|_{2}^2  + \sum_{s\varepsilon I(i)} \left \| y_s  \right \|_{2}^{2} )
   $$

   

# Q3:矩阵分解算法在推荐系统中有哪些应用场景，存在哪些不足？

## A3:

SVD可以对矩阵进行无损分解，在实际中，我们可以抽取前K个特征，对矩阵进行降维。但矩阵分解只考虑了user和item两个特征，未能考虑更多特征维度，实际上一个预测问题包含的特征维度可能很多。

# Q4:假设一个小说网站，有N部小说，每部小说都有摘要描述。如何针对该网站制定基于内容的推荐系统，即用户看了某部小说后，推荐其他相关的小说。原理和步骤是怎样的?

## A4:

1. 对每部小说进行特征抽取，形成每部小说的特征向量；
   + N-Gram，提取N个连续字的集合，作为特征
   + TF-IDF词频-逆向文档率，按照(min_df, max_df)提取关键词，并生成TFIDF矩阵
2. 计算小说之间的相似度矩阵；根据TF-IDF计算它们之间的余弦相似度，得到相似度矩阵；
3. 对指定的小说，基于相似度矩阵进行由大到小排序，推荐相似度最大的Top-k部小说。

# Q5:Word2Vec的应用场景有哪些?

## A5:

Word2Vec通过Embedding，把原先词所在空间映射到一个新的空间中去，使得语义上相似的单词在该空间内距离相近。Word2Vec将待解决的问题转换成为单词word和文章doc的对应关系。

- 它有两种模式：
  - Skip-Gram，给定input word预测上下文
  - CBOW，给定上下文，预测input word（与Skip-Gram相反）

- 应用场景：

  在NLP中，可以用它来寻找相关词、发现新词、命名实体识别、信息索引、情感分析等。

  - 当前用户推荐他/她可能关注的大V： 每一个大V就是一个词，根据每一个用户关注大V的顺序，生成文章
  - 给用户推荐可能感兴趣的商品：每个商品就是一个词，根据用户对商品的行为顺序，生成文章；
  - App 商店中，向用户推荐感兴趣的 App：每个 App 就是一个词；将每个用户下载的 App，按照下载的顺序排列，形成文章；
  - 广告主在媒体网站上打广告，媒体网站提供一个后台管理系统，可以让广告主自行决定要将广告推荐给哪些目标人群：每一个页面就是一个词；将每个用户浏览的页面，按照浏览的顺序排列，形成文章。

