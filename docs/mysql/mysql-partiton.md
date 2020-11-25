---
title: MySQL 分区表
date: 2018-08-14 16:01:01
categories: MySQL
tags: [MySQL, 分区表]
toc: true
---
通过本文学习 MySQL 分区表。
<!-- more -->

## 1 定义

表的分区指根据可以设置为任意大小的规则，跨文件系统分配单个表的多个部分。实际上，表的不同部分在不同的位置被存储为单独的表。用户所选择的、实现数据分割的规则被称为分区函数，这在 MySQL 中它可以是模数，或者是简单的匹配一个连续的数值区间或数值列表，或者是一个内部 HASH 函数，或一个线性 HASH 函数。**当然，表的分区并不是分的越多越好，当表的分区太多时找分区又是一个性能的瓶颈了，建议在 200 个分区以内。**

MySQL 支持 4 种类型的分区表，即 RANGE、LIST、HASH、KEY ，其中 RANGE 和 LIST 类似，按一种区间进行分区， HASH 与 KEY 类似，是按照某种算法对字段进行分区。

![](https://static.oschina.net/uploads/space/2018/0108/113401_hgsL_593078.jpg)

## 2 使用场景

- 某张表的数据量非常大，通过索引已经不能很好的解决查询性能的问题。
- 表的数据可以按照某种条件进行分类，以致于在查询的时候性能得到很大的提升。

## 3 特性

### 3.1 优点

- 对于那些已经失去保存意义的数据，通常可以通过删除与那些数据有关的分区，很容易地删除那些数据。相反地，在某些情况下，添加新数据的过程又可以通过为那些新数据专门增加一个新的分区，来很方便地实现。
- 分区表的数据更容易维护，如：想批量删除大量数据可以使用清除整个分区的方式。另外，还可以对一个独立分区进行优化、检查、修复等操作。如果需要，还可以备份和恢复独立的分区，这在非常大的数据集的场景下效果非常好。
- 一些查询可以得到极大的优化，这主要是借助于满足一个给定 WHERE 语句的数据可以只保存在一个或多个分区内，这样在查找时就不用查找其他剩余的分区。因为分区可以在创建了分区表后进行修改，所以在第一次配置分区方案时还不曾这么做时，可以重新组织数据，来提高那些常用查询的效率。
- 涉及到例如 SUM() 和 COUNT() 这样聚合函数的查询，可以很容易地进行并行处理。这意味着查询可以在每个分区上同时进行，最终结果只需通过总计所有分区得到的结果。 
- 通过跨多个磁盘来分散数据查询，来获得更大的查询吞吐量。
- 分区表的数据可以分布在不同的物理设备上，从而高效地利用多个硬件设备。(PARTITION p0 VALUES LESS THAN (3000000) DATA DIRECTORY = '/data0/data' INDEX DIRECTORY = '/data1/idx')。
- 可以使用分区表来避免某些特殊的瓶颈，如：innodb 的单个索引的互斥访问，ext3文件系统的 inode 锁竞争等。

### 3.2 限制

- 一个表最多只能有 1024 个分区（mysql5.6 之后支持 8192 个分区）。
- 在 mysql5.1 中分区表达式必须是整数，或者是返回整数的表达式，在 5.5 之后，某些场景可以直接使用字符串列和日期类型列来进行分区（使用 varchar 字符串类型列时，一般还是字符串的日期作为分区）。
- 分区表要求分区字段必须是主键或者是主键的一部分（即联合主键）。唯一索引列？?
- mysql 数据库支持的分区类型为水平（横向）分区，并不支持垂直（纵向）分区。因此，mysql 数据库的分区中索引是局部分区索引，一个分区中既存放了数据又存放了索引，而全局分区是指的数据库放在各个分区中，但是所有的数据的索引放在另外一个对象中。
- 目前 mysql 不支持空间类型和临时表类型进行分区。不支持全文索引。
- 对表执行分区操作的进程会占用表的写锁，不影响读，例如在这些分区上的 INSERT 和 UPDATE 操作只有在分区操作完成后才能执行。
- 使用 InnoDB 引擎的分区表不支持外键。
- 分区表不支持 query cache ，在分区表的查询中自动避开了 query cache 。
- 分区表不支持全文索引或者搜索，即使分区表的存储引擎是 InnoDB 或者 MyISAM 也不行。

## 4 使用

### 4.1 验证是否支持分区

```sql
-- YES/NO
SHOW VARIABLES LIKE '%have_partitioning%';
```

### 4.2 分区类型

#### 4.2.1 RANGE

RANGE 分区：基于属于一个给定连续区间的列值，把多行分配给分区。

```sql
-- 订单表分区字段是 atime ，根据 RANGE 分区，这样当你向该表中插入数据的时候， MySQL 会根据 YEAR(atime) 的值进行分区存储
DROP TABLE IF EXISTS `my_orders`;
CREATE TABLE `my_orders` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '表主键',
  `pid` int(10) unsigned NOT NULL COMMENT '产品ID',
  `price` decimal(15,2) NOT NULL COMMENT '单价',
  `num` int(11) NOT NULL COMMENT '购买数量',
  `uid` int(10) unsigned NOT NULL COMMENT '客户ID',
  `atime` datetime NOT NULL COMMENT '下单时间',
  `utime` int(10) unsigned NOT NULL DEFAULT 0 COMMENT '修改时间',
  `isdel` tinyint(4) NOT NULL DEFAULT '0' COMMENT '软删除标识',
  PRIMARY KEY (`id`,`atime`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
-- 分区信息
PARTITION BY RANGE (YEAR(atime))
(
   PARTITION p0 VALUES LESS THAN (2016),
   PARTITION p1 VALUES LESS THAN (2017),
   PARTITION p_other VALUES LESS THAN MAXVALUE
);

-- 检查分区是否创建成功
EXPLAIN PARTITIONS SELECT * FROM `my_orders`

-- 向分区表插入数据
INSERT INTO my_orders(`pid`,`price`,`num`,`uid`,`atime`) VALUES(1,12.23,1,89757,CURRENT_TIMESTAMP());
INSERT INTO my_orders(`pid`,`price`,`num`,`uid`,`atime`) VALUES(1,12.23,1,89757,'2016-05-01 00:00:00');
INSERT INTO my_orders(`pid`,`price`,`num`,`uid`,`atime`) VALUES(1,12.23,1,89757,'2017-05-01 00:00:00');
INSERT INTO my_orders(`pid`,`price`,`num`,`uid`,`atime`) VALUES(1,12.23,1,89757,'2018-05-01 00:00:00');
INSERT INTO my_orders(`pid`,`price`,`num`,`uid`,`atime`) VALUES(1,12.23,1,89756,'2015-05-01 00:00:00');
INSERT INTO my_orders(`pid`,`price`,`num`,`uid`,`atime`) VALUES(1,12.23,1,89756,'2016-05-01 00:00:00');
INSERT INTO my_orders(`pid`,`price`,`num`,`uid`,`atime`) VALUES(1,12.23,1,89756,'2017-05-01 00:00:00');
INSERT INTO my_orders(`pid`,`price`,`num`,`uid`,`atime`) VALUES(1,12.23,1,89756,'2018-05-01 00:00:00');

-- 复制大量数据
INSERT INTO `my_orders`(`pid`,`price`,`num`,`uid`,`atime`) SELECT `pid`,`price`,`num`,`uid`,`atime` FROM `my_orders`;

-- 检查性能：扫描的行数比没用分区表时少，查询效率提高
EXPLAIN PARTITIONS SELECT * FROM `my_orders` WHERE `uid`=89757 AND `atime`< CURRENT_TIMESTAMP();
```

#### 4.2.2 LIST

LIST 分区：类似于按 RANGE 分区，区别在于 LIST 分区是基于列值匹配一个离散值集合中的某个值来进行选择。

```sql
-- LIST 分区和 RANGE 分区很类似
CREATE TABLE `products` (
`id`  bigint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '表主键' ,
`name`  varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '产品名称' ,
`metrial`  tinyint UNSIGNED NOT NULL COMMENT '材质' ,
`weight`  double UNSIGNED NOT NULL DEFAULT 0 COMMENT '重量' ,
`vol`  double UNSIGNED NOT NULL DEFAULT 0 COMMENT '容积' ,
`c_id`  tinyint UNSIGNED NOT NULL COMMENT '供货公司ID' ,
PRIMARY KEY (`id`,`c_id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8
-- 分区信息
PARTITION BY LIST(c_id)
(
    PARTITION pA VALUES IN (1,3,11,13),
    PARTITION pB VALUES IN (2,4,12,14),
    PARTITION pC VALUES IN (5,7,15,17),
    PARTITION pD VALUES IN (6,8,16,18),
    PARTITION pE VALUES IN (9,10,19,20)
);

-- 检查分区是否创建成功
EXPLAIN PARTITIONS SELECT * FROM `products`
```

#### 4.2.3 HASH

HASH 分区：基于用户定义的表达式的返回值来进行选择的分区，该表达式使用将要插入到表中的这些行的列值进行计算。这个函数可以包含 MySQL 中有效的、产生非负整数值的任何表达式。

```sql
-- msgs 表按照 sub_id 进行 HASH 分区，一共分了十个区
CREATE TABLE `msgs` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '表主键',
  `sender` int(10) unsigned NOT NULL COMMENT '发送者ID',
  `reciver` int(10) unsigned NOT NULL COMMENT '接收者ID',
  `msg_type` tinyint(3) unsigned NOT NULL COMMENT '消息类型',
  `msg` varchar(225) NOT NULL COMMENT '消息内容',
  `atime` int(10) unsigned NOT NULL COMMENT '发送时间',
  `sub_id` tinyint(3) unsigned NOT NULL COMMENT '部门ID',
  PRIMARY KEY (`id`,`sub_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
-- 分区信息
PARTITION BY HASH(sub_id)
PARTITIONS 10;

-- 检查分区是否创建成功
EXPLAIN PARTITIONS SELECT * FROM `msgs`
```

#### 4.2.4 KEY

KEY 分区：类似于按 HASH 分区，区别在于 KEY 分区只支持计算一列或多列，且 MySQL 服务器提供其自身的哈希函数。必须有一列或多列包含整数值。

#### 4.2.5 子分区

```sql
CREATE TABLE `msgss` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '表主键',
  `sender` int(10) unsigned NOT NULL COMMENT '发送者ID',
  `reciver` int(10) unsigned NOT NULL COMMENT '接收者ID',
  `msg_type` tinyint(3) unsigned NOT NULL COMMENT '消息类型',
  `msg` varchar(225) NOT NULL COMMENT '消息内容',
  `atime` int(10) unsigned NOT NULL COMMENT '发送时间',
  `sub_id` tinyint(3) unsigned NOT NULL COMMENT '部门ID',
  PRIMARY KEY (`id`,`atime`,`sub_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
-- 分区信息
PARTITION BY RANGE (atime) SUBPARTITION BY HASH (sub_id) 
(
		PARTITION t0 VALUES LESS THAN(1451577600)
		(
			SUBPARTITION s0,
			SUBPARTITION s1,
			SUBPARTITION s2,
			SUBPARTITION s3,
			SUBPARTITION s4,
			SUBPARTITION s5
		),
		PARTITION t1 VALUES LESS THAN(1483200000)
		(
			SUBPARTITION s6,
			SUBPARTITION s7,
			SUBPARTITION s8,
			SUBPARTITION s9,
			SUBPARTITION s10,
			SUBPARTITION s11
		),
		PARTITION t_other VALUES LESS THAN MAXVALUE
		(
			SUBPARTITION s_other1,
			SUBPARTITION s_other2,
			SUBPARTITION s_other3,
			SUBPARTITION s_other4,
			SUBPARTITION s_other5,
			SUBPARTITION s_other6
		)
);

-- 向分区表插入数据
INSERT INTO `msgss`(`sender`,`reciver`,`msg_type`,`msg`,`atime`,`sub_id`) VALUES(1,2,0,'Hello HASH',UNIX_TIMESTAMP(NOW()),1);
INSERT INTO `msgss`(`sender`,`reciver`,`msg_type`,`msg`,`atime`,`sub_id`) VALUES(1,2,0,'Hello HASH 2',UNIX_TIMESTAMP(NOW()),2);
INSERT INTO `msgss`(`sender`,`reciver`,`msg_type`,`msg`,`atime`,`sub_id`) VALUES(1,2,0,'Hello HASH 3',UNIX_TIMESTAMP(NOW()),3);
INSERT INTO `msgss`(`sender`,`reciver`,`msg_type`,`msg`,`atime`,`sub_id`) VALUES(1,2,0,'Hello HASH 10',UNIX_TIMESTAMP(NOW()),10);
INSERT INTO `msgss`(`sender`,`reciver`,`msg_type`,`msg`,`atime`,`sub_id`) VALUES(1,2,0,'Hello HASH 7',UNIX_TIMESTAMP(NOW()),7);
INSERT INTO `msgss`(`sender`,`reciver`,`msg_type`,`msg`,`atime`,`sub_id`) VALUES(1,2,0,'Hello HASH 5',UNIX_TIMESTAMP(NOW()),5);

INSERT INTO `msgss`(`sender`,`reciver`,`msg_type`,`msg`,`atime`,`sub_id`) VALUES(1,2,0,'Hello HASH',1451577607,1);
INSERT INTO `msgss`(`sender`,`reciver`,`msg_type`,`msg`,`atime`,`sub_id`) VALUES(1,2,0,'Hello HASH 2',1451577609,2);
INSERT INTO `msgss`(`sender`,`reciver`,`msg_type`,`msg`,`atime`,`sub_id`) VALUES(1,2,0,'Hello HASH 3',1451577623,3);
INSERT INTO `msgss`(`sender`,`reciver`,`msg_type`,`msg`,`atime`,`sub_id`) VALUES(1,2,0,'Hello HASH 10',1451577654,10);
INSERT INTO `msgss`(`sender`,`reciver`,`msg_type`,`msg`,`atime`,`sub_id`) VALUES(1,2,0,'Hello HASH 7',1451577687,7);
INSERT INTO `msgss`(`sender`,`reciver`,`msg_type`,`msg`,`atime`,`sub_id`) VALUES(1,2,0,'Hello HASH 5',1451577699,5);

INSERT INTO `msgss`(`sender`,`reciver`,`msg_type`,`msg`,`atime`,`sub_id`) VALUES(1,2,0,'Hello HASH',1514736056,1);
INSERT INTO `msgss`(`sender`,`reciver`,`msg_type`,`msg`,`atime`,`sub_id`) VALUES(1,2,0,'Hello HASH 2',1514736066,2);
INSERT INTO `msgss`(`sender`,`reciver`,`msg_type`,`msg`,`atime`,`sub_id`) VALUES(1,2,0,'Hello HASH 3',1514736076,3);
INSERT INTO `msgss`(`sender`,`reciver`,`msg_type`,`msg`,`atime`,`sub_id`) VALUES(1,2,0,'Hello HASH 10',1514736086,10);
INSERT INTO `msgss`(`sender`,`reciver`,`msg_type`,`msg`,`atime`,`sub_id`) VALUES(1,2,0,'Hello HASH 7',1514736089,7);
INSERT INTO `msgss`(`sender`,`reciver`,`msg_type`,`msg`,`atime`,`sub_id`) VALUES(1,2,0,'Hello HASH 5',1514736098,5);

-- 检查分区是否创建成功
EXPLAIN PARTITIONS SELECT * FROM `msgss`
```

**子分区注意事项：**

- 每个子分区的数量必须相同。
- 只要在一个分区表的任何分区上使用 subpartition 来明确定义任何子分区，就必须在所有分区上定义子分区，不能漏掉一些分区不进行子分区。
- 每个 subpartition 子句必须包括子分区的一个名字。
- 子分区的名字必须是唯一的，不能在一张表中出现重名的子分区。
- 子分区必须使用 HASH 或者 KEY 分区。只有 RANG E和 LIST 分区支持被子分区； HASH 和 KEY 不支持被子分区。

### 4.3 新增分区

```sql
-- RANGE 类型
ALTER TABLE `my_orders` ADD PARTITION
(
	PARTITION p2 VALUES LESS THAN (2018)
);

-- HASH/KEY类型。将分区总数扩展到12个
ALTER TABLE `msgs` ADD PARTITION PARTITIONS 12;
```

### 4.4 合并分区

```sql
-- RANGE 类型
ALTER TABLE `my_orders` REORGANIZE PARTITION p0,p1 INTO
(
	PARTITION p0_p1 VALUES LESS THAN (2017)
);

-- LIST类型
ALTER TABLE `products` REORGANIZE PARTITION pA,pB INTO
(
	PARTITION pA_pB VALUES IN (1,3,11,13,2,4,12,14)
);

-- HASH/KEY 类型。在这里数量只能比原来少不能多，想要增加可以用 ADD PARTITION 方法
ALTER TABLE `msgs` REORGANIZE PARTITION COALESCE PARTITION 8;
```

### 4.5 删除分区

```sql
-- 先删除。当删除了一个分区，也同时删除了该分区中所有的数据
ALTER TABLE `my_orders` DROP PARTITION p0;

-- 再重建，数据重新组合
ALTER TABLE `my_orders` PARTITION BY RANGE (YEAR(atime)) (......);
```

## 5 扩展阅读

姜海强 - MySQL：[http://blog.csdn.net/jhq0113/article/category/2897729](http://blog.csdn.net/jhq0113/article/category/2897729)

---

- [MySQL 教程合集](https://mp.weixin.qq.com/s/jflrWU62pBtevS62lEIHkQ)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)