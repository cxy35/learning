Elasticsearch 概述。
<!-- more -->

## 1 简介

Elasticsearch 是一个基于 Lucene 的搜索服务器。它提供了一个分布式的全文搜索引擎，基于 restful web 接口。Elasticsearch 是用 Java 语言开发的，基于 Apache 协议的开源项目，是目前最受欢迎的企业搜索引擎。Elasticsearch 广泛运用于云计算中，能够达到实时搜索，具有稳定，可靠，快速的特点。

## 2 基本概念

- `Near Realtime`（近实时）：Elasticsearch 是一个近乎实时的搜索平台，这意味着从索引文档到可搜索文档之间只有一个轻微的延迟(通常是一秒钟)。
- Cluster（集群）：集群是一个或多个节点的集合，它们一起保存整个数据，并提供跨所有节点的联合索引和搜索功能。每个集群都有自己的唯一集群名称，节点通过名称加入集群。
- `Node`（节点）：节点是指属于集群的单个 Elasticsearch 实例，存储数据并参与集群的索引和搜索功能。可以将节点配置为按集群名称加入特定集群，默认情况下，每个节点都设置为加入一个名为 elasticsearch 的集群。
- `Index`（索引）：索引是一些具有相似特征的文档集合，**类似于 MySQL 中数据库的概念**。
- `Type`（类型）：类型是索引的逻辑类别分区，通常，为具有一组公共字段的文档类型，**类似于 MySQL 中表的概念**。注意：由于一些原因，在 Elasticsearch 6.0 以后，一个 Index 只能含有一个 Type。这其中的原因是：相同 index 的不同映射 type 中具有相同名称的字段是相同；在 Elasticsearch 索引中，不同映射 type 中具有相同名称的字段在 Lucene 中被同一个字段支持。在默认的情况下是 `_doc` 。在未来 8.0 的版本中，type 将被彻底删除。
- `Document`（文档）：文档是可被索引的基本信息单位，以 JSON 形式表示，**类似于 MySQL 中行的概念**。
- `Field`（字段）：**类似于 MySQL 中列的概念**。
- `Shards`（分片）：当索引存储大量数据时，可能会超出单个节点的硬件限制，为了解决这个问题，Elasticsearch 提供了将索引细分为分片的概念。分片机制赋予了索引水平扩容的能力，并允许跨分片分发和并行化操作，从而提高性能和吞吐量。
- `Replicas`（副本）：在可能出现故障的网络环境中，需要有一个故障切换机制，Elasticsearch 提供了将索引的分片复制为一个或多个副本的功能，副本在某些节点失效的情况下提供高可用性。

---

|Elasticsearch|数据库|
|:-|:-|
|索引 Index|数据库 Database|
|类型 Type|表 Table|
|文档 Document|行 Row|
|字段 Field|列 Column|

## 3 安装

详情见 [Elasticsearch 安装](https://www.cxy35.top/#/docs/elastic/elasticsearch-install)

## 4 常用命令

详情见 [Elasticsearch 常用命令](https://www.cxy35.top/#/docs/elastic/elasticsearch-command)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.top](https://cxy35.top)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)