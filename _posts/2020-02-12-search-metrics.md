---
layout:     post
title:      "「笔记」指标计算"
subtitle:	" 信息检索指标计算逻辑"
date:       2020-02-21 20:00:00
author:     "cuiming"
header-img: "img/search_metrics.jpg"
catalog: true
tags:	

   - 指标计算
   - NLP
---

#### <b><font color='LightSeaGreen'>1. MAP(Mean Average Precision),平均正确率</font></b>

##### 1.1 计算方式

MAP指标能从整体上反映模型的检索性能,单次搜索的正确率(AP)计算如下：
$$
AP=\frac{\sum_{k=1}^{n}P(k)\times rel(k)}{相关文档个数} \tag{1}
$$
其中，$P(k)$为前k个结果的准确率,$rel(k)$表示第k个文档是否相关，相关则为1，不相关为0，所以多次搜索结果求平均即为MAP,
$$
MAP=\frac{\sum_{q=1}^{Q}AP_{i}}{Q} \tag{2}
$$

其中Q为查询的个数。

##### 1.2 使用示例

假设query1搜索返回了6个文档，每个文档的相关情况为1，1，0，1，0，1

假设query2搜索返回4个文档，每个文档的相关情况为1，0，1，1，则MAP的计算方式为：

<center>

$$
AP_{1} = \frac{1\times1 + 1\times1+0.75\times1+0.67\times1}{4}=0.855\\
AP_{2} = \frac{1\times1+0.67\times1+0.75\times1}{3}=0.806\\
MAP= \frac{AP_{1}+AP_{2}}{2} = 0.83
$$

</center>

##### 1.3 缺点

MAP指标只区分了相关和不相关，而没有相关度的差异。

#### <b><font color='LightSeaGreen'>2. CG（Cumulative Gain），累积增益</font></b>

##### 2.1 计算方式

$$
CG@K = \sum_{i=1}^{k}rel_{i} \tag{3}
$$

其中$rel_i$表示第$i$个搜索结果的相关程度，相关程度可以根据自己业务情况定义,比如

| 相关程度 | 值   |
| -------- | ---- |
| Good     | 3    |
| Fair     | 2    |
| Simple   | 1    |
| Bad      | 0    |

##### 2.2 使用示例

假设query1搜索返回了6个文档，每个文档的相关情况为2，1，0，3，0，1

假设query2搜索返回4个文档，每个文档的相关情况为3，0，1，2

<center>

$$
CG@6_{1} = 2+1+3+1 = 7\\
CG@6_{2} = 3+1+2 = 6\\
CG = \frac{CG@6_{1}+CG@6_{2}}{2} = 6.5
$$

</center>

##### 2.3 不足

CG虽然增加了相关程度的权重，但是因为是权重累积，所以忽略了排序位置的影响，比如对于搜索排序结果(3, 1,0)、(0,1,3)对于CG指标而言结果都是一样的，这显然是不合理的。

#### <b><font color='LightSeaGreen'>3. DCG（Discounted Cumulative Gain），折扣累积增益</font></b>

##### 3.1 计算方式

DCG在CG的基础上增加了位置信息的计算，计算方式如下：
$$
DCG@K=\sum_{i=1}^{k}\frac{rel_i}{log_{2}(i+1)} \tag{4}
$$


##### 3.2 使用示例

假设query1搜索返回了6个文档，每个文档的相关情况为2，1，0，3，0，1

假设query2搜索返回4个文档，每个文档的相关情况为3，0，1，2

<center>

$$
DCG@6_{1}=\frac{2}{log_{2}{2}}+\frac{1}{log_{2}{3}}+\frac{3}{log_{2}{5}}+\frac{1}{log_{2}{7}}=4.28\\
DCG@6_{2}=\frac{3}{log_{2}{2}}+\frac{1}{log_{2}{4}}+\frac{2}{log_{2}{5}}=3.86\\
DCG@6 = \frac{DCG@6_{1}+DCG@6_{2}}{2} = 4.07
$$

</center>

##### 3.3 不足

DCG虽然引入了位置损失，但是不同的搜索结果对应的文档个数，以及分值都不一致，所以DCG的取值范围是动态变化的，这就对于结果的好还是不好给出明确的判定。

#### <b><font color='LightSeaGreen'>4. NDCG（Normalized DCG），归一化折扣累积增益</font></b>

##### 4.1 计算方式

既然要做归一化，那拿什么做归一化呢？这里就引入IDCG的概念，IDCG其实就是把排序结果按照相关度降序排列后重新计算的DCG。这样就可以把指标归一化到0-1的范围内，约接近1说明约接近理想排序效果。
$$
NDCG@K=\frac{DCG@K}{IDCG@K} \tag{5}
$$

##### 4.2 使用示例

假设query1搜索返回了6个文档，每个文档的相关情况为2，1，0，3，0，1

假设query2搜索返回4个文档，每个文档的相关情况为3，0，1，2

<center>

$$
IDCG@6_{1} =\frac{3}{log_{2}2}+\frac{2}{log_{2}3}+\frac{1}{log_{2}4}+\frac{1}{log_{2}5}=5.19\\
IDCG@6_{2} = \frac{3}{log_{2}2}+\frac{2}{log_{2}3}+\frac{1}{log_{2}4} = 4.76\\
NDCG@6_{1} = \frac{DCG@6_{1}}{IDCG@6_{1}} =\frac{4.28}{5.19}=0.82 \\
NDCG@6_{2} = \frac{DCG@6_{2}}{IDCG@6_{2}} =\frac{3.86}{4.76}=0.81\\
NDCG = \frac{NDCG@6_{1} +NDCG@6_{2}}{2}=0.815
$$

</center>




