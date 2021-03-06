---
title: 7.ES分布式集群
date: 2020-03-03 22:03:00
tags:
    - Elaticsearch
categories:
    - 中间件
    - Elaticsearch
---

## 7.1 分布式特性

Elasticsearch 的分布式架构带来的好处:

- 存储的⽔平扩容，⽀持 PB 级数据
- 提⾼系统的可⽤性，部分节点停⽌服务，整个集群的服务不受影响

## 7.2 集群节点类型

- **Master Eligible Node**:可以被选为主节点的节点

- **Voting-only master-eligible node**:只能投票选择主节点的节点

- **Data nodes:**数据节点

- **Coordinating node**:协调调度节点

- **Ingest nodes**: 流水线节点

- **Machine learning node**:机器学习节点



### 7.2.1 Master Node

- Master Node 的职责:
	- **处理创建，删除索引等请求 /决定分⽚被分配到哪个节点 / 负责索引的创建与删除**
	- **维护并且更新 Cluster State**


- Master Node 的最佳实践:
  - Master 节点⾮常重要，在部署上需要考虑解决单点的问题
  - 为⼀个集群设置多个 Master 节点 / 每个节点只承担 Master 的单⼀⻆⾊



### 7.2.2 Master Eligible Nodes

- ⼀个集群，⽀持配置多个 Master Eligible 节点。这些节点可以在必要时(如 Master 节点出

现故障，⽹络故障时)参与选主流程，成为 Master 节点

- 每个节点启动后，默认就是⼀个 Master eligible 节点
  - 可以设置 node.master: false 禁⽌
- 当集群内第⼀个 Master eligible 节点启动时候，它会将⾃⼰选举成 Master 节点

### 7.2.3 Data Node

- 可以保存数据的节点，叫做 Data Node,节点启动后，默认就是数据节点。可以设置 node.data: false 禁⽌
- Data Node的职责:保存分⽚数据。在数据扩展上起到了⾄关重要的作⽤（由 Master Node 决定如何把分⽚分发到数据节点上）
-  通过增加数据节点可以解决数据⽔平扩展和解决数据单点问题



## 7.3 集群架构图

![](http://dist415.oss-cn-beijing.aliyuncs.com/escluster.png)



## 7.4 集群健康状态

健康状态针对一个索引，Elasticsearch 中其实有专门的衡量索引健康状况的标志，分为三个等级：

- **green，绿色**。**这代表所有的主分片和副本分片都已分配。你的集群是 100% 可用的**
- **yellow，黄色**。**所有的主分片已经分片了，但至少还有一个副本是缺失的**。不会有数据丢失，所以搜索结果依然是完整的。不过，你的高可用性在某种程度上被弱化。如果更多的分片消失，你就会丢数据了。所以可把 yellow 想象成一个需要及时调查的警告。
- **red，红色**。**至少一个主分片以及它的全部副本都在缺失中**。这意味着你在缺少数据：搜索只能返回部分数据，而分配到这个分片上的写入请求会返回一个异常。

## 7.5 集群选主过程

![](http://dist415.oss-cn-beijing.aliyuncs.com/esmaster.png)

## 7.6 如何避免脑裂问题

- 限定⼀个选举条件，设置 quorum(仲裁)，只有在 Master eligible 节点数⼤于 quorum 时，才能进⾏选举
  - Quorum = （master 节点总数 /2）+ 1 
  - 当 3 个 master eligible 时，设置 discovery.zen.minimum_master_nodes 为 2，即可避免脑裂

- 从 7.0 开始，⽆需这个配置
	- 移除 minimum_master_nodes 参数，让Elasticsearch⾃⼰选择可以形成仲裁的节点。
	-  典型的主节点选举现在只需要很短的时间就可以完成。集群的伸缩变得更安全、更容易，并且可能造成丢失数据的系统配置选项更少了。
	-  节点更清楚地记录它们的状态，有助于诊断为什么它们不能加⼊集群或为什么⽆法选举出主节点








