---
title: 1.ES简介
date: 2020-03-02 21:00:00
tags:
    - Elaticsearch
categories:
    - 中间件
    - Elaticsearch
---

## 1.1 介绍

Elaticsearch，简称为es， es是一个开源的**高扩展的分布式全文检索引擎**，它可以近**乎实时的存储、检索数据**；本身扩展性很好，可以扩展到上百台服务器，处理PB级别（大数据时代）的数据。es也使用Java开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单**RESTful API**来隐藏Lucene的复杂性，从而让全文搜索变得简单。
据国际权威的数据库产品评测机构DB Engines的统计，在2016年1月，ElasticSearch已超过Solr等，**成为排名第一的搜索引擎类应用**

## 1.2 功能

### 1.2.1 分布式的搜索引擎和数据分析引擎

搜索：百度，网站的站内搜索，IT系统的检索
数据分析：电商网站，最近7天牙膏这种商品销量排名前10的商家有哪些；新闻网站，最近1个月访问量排名前3的新闻版块是哪些
分布式，搜索，数据分析

### 1.2.2 全文检索，结构化检索，数据分析

**全文检索**：我想搜索商品名称包含牙膏的商品，select * from products where product_name like "%牙膏%"
**结构化检索**：我想搜索商品分类为日化用品的商品都有哪些，`select * from products where category_id='日化用品'`

部分匹配、自动完成、搜索纠错、搜索推荐

**数据分析**：我们分析每一个商品分类下有多少个商品，select category_id,count(*) from products group by category_id

### 1.2.3 对海量数据进行近实时的处理

**分布式**：ES自动可以将海量数据分散到多台服务器上去存储和检索
海联数据的处理：分布式以后，就可以采用大量的服务器去存储和检索数据，自然而然就可以实现海量数据的处理了
**近实时**：检索个数据要花费1小时（这就不要近实时，离线批处理，batch-processing）；在秒级别对数据进行搜索和分析

跟分布式/海量数据相反的：lucene，单机应用，只能在单台服务器上使用，最多只能处理单台服务器可以处理的数据量

## 1.3 主要概念

Elasticsearch的主要概念如下:

- **节点** - 它指的是Elasticsearch的单个正在运行的实例。单个物理和虚拟服务器容纳多个节点，这取决于其物理资源的能力，如RAM，存储和处理能力。
- **集群** - 它是一个或多个节点的集合。 集群为整个数据提供跨所有节点的集合索引和搜索功能。
- **索引** - 它是不同类型的文档和文档属性的集合。索引还使用分片的概念来提高性能。 例如，一组文档包含社交网络应用的数据。
- **类型/映射** - 它是共享同一索引中存在的一组公共字段的文档的集合。 例如，索引包含社交网络应用的数据，然后它可以存在用于用户简档数据的特定类型，另一类型可用于消息的数据，以及另一类型可用于评论的数据。
- **文档** - 它是以JSON格式定义的特定方式的字段集合。每个文档都属于一个类型并驻留在索引中。每个文档都与唯一标识符(称为UID)相关联。
- **碎片** - 索引被水平细分为碎片。这意味着每个碎片包含文档的所有属性，但包含的数量比索引少。水平分隔使碎片成为一个独立的节点，可以存储在任何节点中。主碎片是索引的原始水平部分，然后这些主碎片被复制到副本碎片中。
- **副本** - Elasticsearch允许用户创建其索引和分片的副本。 复制不仅有助于在故障情况下增加数据的可用性，而且还通过在这些副本中执行并行搜索操作来提高搜索的性能。

## 1.4 优点

Elasticsearch的优点:

- Elasticsearch是基于Java开发的，这使得它在几乎每个平台上都兼容。
- Elasticsearch是实时的，换句话说，一秒钟后，添加的文档可以在这个引擎中搜索得到
- Elasticsearch是分布式的，这使得它易于在任何大型组织中扩展和集成。
- 通过使用Elasticsearch中的网关概念，创建完整备份很容易。
- 与Apache Solr相比，在Elasticsearch中处理多租户非常容易。
- Elasticsearch使用JSON对象作为响应，这使得可以使用不同的编程语言调用Elasticsearch服务器
- Elasticsearch支持几乎大部分文档类型，但不支持文本呈现的文档类型。

## 1.5 缺点

Elasticsearch的缺点:

- Elasticsearch在处理请求和响应数据方面没有多语言和数据格式支持(仅在JSON中可用)，与Apache Solr不同，Elasticsearch不可以使用CSV，XML等格式

## 1.6 特点

- 可以作为一个大型分布式集群（数百台服务器）技术，处理PB级数据，服务大公司；也可以运行在单机上，服务小公司
- Elasticsearch不是什么新技术，主要是将全文检索、数据分析以及分布式技术，合并在了一起，才形成了独一无二的ES；lucene（全文检索），商用的数据分析软件（也是有的），分布式数据库（mycat）
- 对用户而言，是开箱即用的，非常简单，作为中小型的应用，直接3分钟部署一下ES，就可以作为生产环境的系统来使用了，数据量不大，操作不是太复杂
- 数据库的功能面对很多领域是不够用的（事务，还有各种联机事务型的操作）；特殊的功能，比如全文检索，同义词处理，相关度排名，复杂数据分析，海量数据的近实时处理；Elasticsearch作为传统数据库的一个补充，提供了数据库所不不能提供的很多功能