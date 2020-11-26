---
title: MySQL 核心知识点
date: 2018-11-12 09:16:23
categories: MySQL
tags: [MySQL]
toc: true
---
本文记录 MySQL 相关的一些核心知识点。
<!-- more -->

## 1 存储引擎（MyISAM 和 InnoDB）

**MyIASM 特点：** 

1. MyISAM没有提供对数据库事务的支持。
2. 不支持行级锁和外键。
3. 由于 2，导致当执行 INSERT 插入或 UPDATE 更新语句时，即执行写操作需要锁定整个表，所以会导致效率降低。
4. MyISAM 保存了表的行数，当执行 SELECT COUNT(*) FROM TABLE 时，可以直接读取相关值，不用全表扫描，速度快。

**InnoDB 特点：** 

1. 支持事务。
2. 支持 4 个级别的事务隔离。
3. 支持多版本读。
4. 支持行级锁。
5. 读写阻塞与事务隔离级别相关。
6. 支持缓存，既能缓存索引，也能缓存数据。
7. 整个表和主键以 Cluster 方式存储，组成一颗平衡树。

**MyISAM 和 InnoDB 的区别：**

1. MyISAM 是非事务安全的，而 InnoDB 是事务安全的。
2. MyISAM 锁的粒度是表级的，而 InnoDB 支持行级锁。
3. MyISAM 支持全文类型索引，而 InnoDB 不支持全文索引，一般我们要借助于 Solr 或者 ES 等来做全文索引。

**使用场景：**

1. 如果要执行大量 select 操作，应该选择 MyISAM 。
2. 如果要执行大量 insert 和 update 操作，应该选择 InnoDB 。
3. 大尺寸的数据集趋向于选择 InnoDB 引擎，因为它支持事务处理和故障恢复。数据库的大小决定了故障恢复的时间长短，InnoDB可以利用事务日志进行数据恢复，这会比较快。主键查询在InnoDB引擎下也会相当快，不过需要注意的是如果主键太长也会导致性能问题。

相对来说，InnoDB 在互联网公司使用更多一些。

## 2 事务隔离级别

事务的 ACID 特性：

- 原子性（Atomicity）
- 一致性（Consistency）
- 隔离性（Isolation）
- 持久性（Durability）

四个特性中最复杂的，莫过于隔离性了。SQL 标准的事务隔离级别：

- serializable/串行执行
- repeatable read/可重复读
- read committed/读提交
- read uncommitted/读未提交

**serializable：**

如果隔离级别为 serializable，则用户之间通过一个接一个顺序地执行当前的事务，这种隔离级别提供了事务之间最大限度的隔离，当然效率也是最低的。

**repeatable read：**

事务不会被看成是一个序列。不过，当前正在执行事务的变化仍然不能被外部看到，也就是说，如果用户在另外一个事务中执行同条 SELECT 语句数次，结果总是相同的。（因为正在执行的事务所产生的数据变化不能被外部看到）。

**read committed：**

安全性比 repeatable read 隔离级别的安全性要差。处于 read committed 级别的事务可以看到其他事务对数据的修改。也就是说，在事务处理期间，如果其他事务修改了相应的表，那么同一个事务的多个 SELECT 语句可能返回不同的结果。

**read uncommitted：**

提供了事务之间最小限度的隔离。除了容易产生虚幻的读操作和不能重复的读操作外，处于这个隔离级的事务可以读到其他事务还没有提交的数据，如果这个事务使用其他事务不提交的变化作为计算的基础，然后那些未提交的变化被它们的父事务撤销，这就导致了大量的数据变化。不过，这种隔离级别从效率上来说，却是最高的。

**MySQL 默认的隔离级别则是 repeatable read**，Oracle 默认的隔离级别是 read committed。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)