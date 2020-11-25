---
title: MySQL 索引
date: 2018-08-12 21:27:48
categories: MySQL
tags: [MySQL, 索引]
toc: true
---
索引是快速搜索的关键，索引的建立对于 MySQL 的高效运行很重要。通过本文学习 MySQL 索引。
<!-- more -->
  
## 1 索引类型 

### 1.1 普通索引

```sql
ALTER TABLE `t_user` ADD INDEX `idx_user_username` (`username`);
```

### 1.2 唯一索引

唯一索引的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。

```sql
ALTER TABLE `t_user` ADD UNIQUE `idx_user_username` (`username`);
```

### 1.3 主键索引

主键索引是一种特殊的唯一索引，不允许有空值，一般是在建表的时候同时创建主键索引。

```sql
CREATE TABLE `t_user` (
    `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键id',
    `username` varchar(16) NOT NULL COMMENT '用户名',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='用户';
```

### 1.4 组合索引

```sql
CREATE TABLE mytable(  
    ID INT NOT NULL,   
    username VARCHAR(16) NOT NULL,  
    city VARCHAR(50) NOT NULL,  
    age INT NOT NULL 
);
```

为了进一步榨取MySQL的效率，就要考虑建立组合索引。就是将 username, city, age 建到一个索引里。

```sql
ALTER TABLE mytable ADD INDEX name_city_age (username(10), city, age);
```

建表时，usernname 长度为 16，这里用 10。这是因为一般情况下名字的长度不会超过 10，这样会加速索引查询速度，还会减少索引文件的大小，提高 INSERT 的更新速度。建立这样的组合索引，其实是相当于分别建立了下面三组组合索引：  

- usernname, city, age
- usernname, city
- usernname 

为什么没有 city, age 这样的组合索引呢？这是因为 MySQL 组合索引“最左前缀”的结果。简单的理解就是只从最左面的开始组合。并不是只要包含这三列的查询都会用到该组合索引，下面的几个 SQL 就会用到这个组合索引：

```sql
SELECT * FROM mytable WHREE username="admin" AND city="郑州"
SELECT * FROM mytable WHREE username="admin"
```

而下面几个则不会用到：  

```sql
SELECT * FROM mytable WHREE age=20 AND city="郑州"
SELECT * FROM mytable WHREE city="郑州"
```

如果分别在 usernname, city, age 上建立单列索引，让该表有3个单列索引，查询时和上述的组合索引效率也会大不一样，远远低于我们的组合索引。虽然此时有了三个索引，但 MySQL 只能用到其中的那个它认为似乎是最有效率的单列索引。

## 2 建立索引的时机

到这里我们已经学会了建立索引，那么我们需要在什么情况下建立索引呢？

一般来说，在 WHERE 和 JOIN 中出现的列需要建立索引，但也不完全如此，因为 MySQL 只对 <，<=，=，>，>=，BETWEEN，IN，以及某些时候的 LIKE 才会使用索引。

```sql
SELECT t.username
FROM mytable t LEFT JOIN mytable2 m
ON t.username=m.username WHERE m.age=20 AND m.city='郑州';
```

```sql
-- 会使用索引
SELECT * FROM mytable WHERE username like 'admin%';
-- 不会使用索引（以通配符 % 和 _ 开头）
SELECT * FROM mytable WHERE username like '%admin';
```
  
## 3 索引的不足之处
 
- 虽然索引大大提高了查询速度，同时却会降低更新表的速度，如对表进行 INSERT/UPDATE/DELETE。因为更新表时，MySQL 不仅要保存数据，还要保存索引文件。
- 建立索引会占用磁盘空间的索引文件。一般情况这个问题不太严重，但如果你在一个大表上创建了多种组合索引，索引文件的会膨胀很快。

索引只是提高效率的一个因素，如果你的 MySQL 有大数据量的表，就需要花时间研究建立最优秀的索引，或优化查询语句。

## 4 使用索引的注意事项
  
- 索引不会包含有 NULL 值的列。

只要列中包含有 NULL 值都将不会被包含在索引中，复合索引中只要有一列含有 NULL 值，那么这一列对于此复合索引就是无效的。所以我们在数据库设计时不要让字段的默认值为 NULL。

- 使用短索引

对串列进行索引，如果可能应该指定一个前缀长度。例如，如果有一个 CHAR(255) 的列，如果在前 10 个或 20 个字符内，多数值是惟一的，那么就不要对整个列进行索引。短索引不仅可以提高查询速度而且可以节省磁盘空间和 I/O 操作。

- 索引列排序

MySQL 查询只使用一个索引，因此如果 where 子句中已经使用了索引的话，那么 order by 中的列是不会使用索引的。因此数据库默认排序可以符合要求的情况下不要使用排序操作；尽量不要包含多个列的排序，如果需要最好给这些列创建复合索引。

- like语句操作

一般情况下不鼓励使用 like 操作，如果非使用不可，如何使用也是一个问题。`like '%aaa%'` 不会使用索引而 `like 'aaa%'` 可以使用索引。

- 不要在列上进行运算

```sql
select * from users where YEAR(adddate) < 2007;
-- 上述语句将在每个行上进行运算，这将导致索引失效而进行全表扫描，因此我们可以改成：
select * from users where adddate < '2007-01-01';
```

- 不使用 NOT IN 和 <> 操作

---

- [MySQL 教程合集](https://mp.weixin.qq.com/s/jflrWU62pBtevS62lEIHkQ)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)