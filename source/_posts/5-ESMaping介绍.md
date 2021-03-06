---
title: 5.ESMapping介绍
date: 2020-03-03 22:00:00
tags:
    - Elaticsearch
categories:
    - 中间件
    - Elaticsearch
---

## 5.1 什么是Mapping


- Mapping类似数据库中的schema的定义，作用如下
  - 定义索引中的字段的名称
  - 定义字段的数据类型，例如字符串，数字，布尔……
  	- 字段，倒排索引的相关配置，(Analyzed r Nt Analyzed,Analyzer)
- Mapping会把JSN文档映射成Lucene所需要的扁平格式
- —个Mapping属于一个索引的Type
  - 每个文档都属于一个Type
  - —个Type有一个Mapping定义
  - 7.0开始，不需要在Mapping定义中指定type信息



## 5.2 字段的数据类型

- 简单类型

  - Text / Keyword

  - Date

  - Integer / Floating

  - Boolean

  - IPv4 & IPv6
- 复杂类型-对象和嵌套对象
  - 对象类型/嵌套类型
- 特殊类型
  - geo_point & geo_shape / percolator



## 5.3 什么是Dynamic Mapping

- 在写入文档时候，如果索引不存在,会自动创建索引
- Dynamic Mapping的机制，使得我们无需手动定义Mappings
- Elasticsearch会自动根据文档信息，推算出字段的类型
- 但是有时候会推算的不对，例如地理位置信息
- 当类型如果设置不对时，会导致一些功能无法正常运行，例如Range查询

## 5.4 类型的自动识别

![](http://dist415.oss-cn-beijing.aliyuncs.com/esmapping.png)



## 5.5 能否更改Mapping的字段类型

### 5.5.1 新增加字段

- Dynamic设为true时，一旦有新增字段的文档写入，Mapping也同时被
- Dynamic设为false, Mapping不会被更新，新增字段的数据无法被索引,但是信息会出现在_source中
- Dynamic设置成Strict,文档写入失败

![](http://dist415.oss-cn-beijing.aliyuncs.com/esdynamic.png)



### 5.5.2 对已有字段

**一旦已经有数据写入，就不再支持修改字段定义**
**Lucene实现的倒排索引，一旦生成后，就不允许修改!如果希望改变字段类型，必须Reindex API,重建索引**

原因

- 如果修改了字段的数据类型，会导致已被索引的属于无法被搜索
- 但是如果是增加新的字段，就不会有这样的影响