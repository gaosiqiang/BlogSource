---
title: 6.ES算法
date: 2020-03-03 22:01:00
tags:
    - Elaticsearch
categories:
    - 中间件
    - Elaticsearch
---

## 6.1 相关性

搜索的相关性算分，描述了一个**文档和査询语句匹配的程度**。ES会对每个匹配查询条件的结果进行算分_score

**打分的本质是排序，需要把最符合用户需求的文档排在前面。**

**ES5之前，默认的相关性算分采用TF-IDF,现在采用BM 25**



![](http://dist415.oss-cn-beijing.aliyuncs.com/esrele.png)



## 6.2 词频 TF

- **Term Frequency：检索词在一篇文档中出现的频率**
  - **检索词出现的次数除以文档的总字数**
- 度量一条查询和结果文档相关性的简单方法：简单将搜索中**每一个词的TF进行相加**
  - TF（区块链）+ TF（的）+ TF（应用）
- Stop Word
  - “的”在文档中出现了很多次，但是对贡献相关度几乎没有用处，不应该考虑他们的TF



## 6.3 	逆⽂档频率 IDF

- DF：检索词索词在所有文档中出现的频率
  - “区块链”在相对比较少的文档中出现
  - “应用”在相对比较多的文档中出现
  - “Stop Word”在大量的文档中出现
- Inverse Document Frequency :**简单说=log（全部文档数/検索词出现过的文档总数）**
- TF-IDF本质上就是将TF求和变成了加权求和
  - ```TF（区块链）*IDF（区块链）+ TF（的）*IDF（的）+ TF（应用）*IDF（应用）```

![](http://dist415.oss-cn-beijing.aliyuncs.com/esidf.png)

## 6.4 TF-IDF 评分公式

![](http://dist415.oss-cn-beijing.aliyuncs.com/estf-idf.png)

## 6.5 BM25

- 从ES5开始，默认算法改为BM 25
- 和经典的TF-IDF相比，当TF无限增加时,BM 25算分会趋于一个数值

![](http://dist415.oss-cn-beijing.aliyuncs.com/esbm25.png)

