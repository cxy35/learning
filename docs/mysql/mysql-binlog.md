---
title: MySQL binlog 详解
date: 2020-07-11 09:59:30
categories: MySQL
tags: [MySQL, binlog]
toc: true
---
通过本文学习 MySQL 中 binlog 相关的内容，包括概述、配置及常用操作、数据恢复实战等。
<!-- more -->

## 1 概述

MySQL 的二进制日志 binlog 可以说是 MySQL 最重要的日志，它记录了所有的 DDL 和 DML 语句（除了数据查询语句 select ），以事件形式记录，还包含语句所执行的消耗的时间，MySQL 的二进制日志是事务安全型的。

- DDL：Data Definition Language 数据库定义语言。主要的命令有 `create、alter、drop` 等，ddl 主要是用在定义或改变表（table）结构、数据类型、表之间的连接和约束等初始工作上，他们大多在建表时候使用。

- DML：Data Manipulation Language 数据操纵语言。主要命令是 `select/update/insert/delete` ，就像它的名字一样，这 4 条命令是用来对数据库里的数据进行操作的语言。

二进制日志包括两类文件：二进制日志索引文件（文件名后缀为 `.index` ，如 `mysql-bin.index`），用于记录所有的二进制文件；二进制日志文件（文件名后缀为 `.00000*` ，如 `mysql-bin.000001`），用于记录数据库所有的 DDL 和 DML （除了数据查询）语句事件。

---

binlog 日志两个最重要的使用场景：主从同步、数据恢复。

**一般来说开启 binlog 日志大概会有 1% 的性能损耗。**

## 2 配置及常用操作

### 2.1 配置 binlog 日志

```bash
vi /usr/local/mydata/etc/my.cnf

[mysqld]
# 必须
log_bin=/usr/local/mydata/log/mysql-bin.log
binlog_format=MIXED
server_id=1

# 非必须
sync_binlog = 100
binlog_cache_size = 1G
max_binlog_cache_size = 1G
max_binlog_size = 1G
expire_logs_days = 7
```

### 2.2 验证 binlog 日志是否开启

```bash
mysql> show variables like 'log_%';
+----------------------------------------+---------------------------------------+
| Variable_name                          | Value                                 |
+----------------------------------------+---------------------------------------+
| log_bin                                | ON                                    |
| log_bin_basename                       | /usr/local/mydata/log/mysql-bin       |
| log_bin_index                          | /usr/local/mydata/log/mysql-bin.index |
| log_bin_trust_function_creators        | OFF                                   |
| log_bin_use_v1_row_events              | OFF                                   |
| log_error                              | /usr/local/mydata/log/alert.log       |
| log_output                             | FILE                                  |
| log_queries_not_using_indexes          | OFF                                   |
| log_slave_updates                      | ON                                    |
| log_slow_admin_statements              | OFF                                   |
| log_slow_slave_statements              | OFF                                   |
| log_throttle_queries_not_using_indexes | 0                                     |
| log_warnings                           | 1                                     |
+----------------------------------------+---------------------------------------+
13 rows in set (0.00 sec)
```

看到 `log_bin=ON`，表示开启成功。

### 2.3 查看所有 binlog 日志列表

```bash
mysql> show binary logs;
+------------------+------------+
| Log_name         | File_size  |
+------------------+------------+
| mysql-bin.000001 | 1678110651 |
| mysql-bin.000002 |   62841042 |
+------------------+------------+
2 rows in set (0.00 sec)
```

如果搭建了主从，还可以使用下列命令。

```bash
mysql> show master logs;
+------------------+------------+
| Log_name         | File_size  |
+------------------+------------+
| mysql-bin.000001 | 1678110651 |
| mysql-bin.000002 |   63933358 |
+------------------+------------+
2 rows in set (0.00 sec)
```

查看 master 状态，即最后（最新）一个 binlog 日志的编号名称，及其最后一个操作事件 pos 结束点（Position）值

```bash
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000002 | 80981731 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

### 2.4 刷新 log 日志

```bash
mysql> flush logs;
```

自此刻开始产生一个新编号的 binlog 日志文件。每当 MySQL 服务重启时，会自动执行此命令，刷新 binlog 日志。另外在 `mysqlddump` 备份数据时加 `-F` 选项也会刷新 binlog 日志。

### 2.5 清除日志文件到指定文件

```bash
mysql> purge master logs to 'mysql-bin.000008';
```

会删除编号之前的日志文件。

**注意：该命令在生产环境下不要轻易执行，特别是要清除的文件较多时，会影响 MySQL 性能。**

### 2.6 重置（清空）所有 binlog 日志

```bash
mysql> reset master;
```

### 2.7 查看 binlog 日志（使用 mysqlbinlog）

```bash
# 按时间范围查询，分页查看
/usr/local/mysql/bin/mysqlbinlog --no-defaults --database=mydb --start-datetime="2020-07-09 10:00:00" --stop-datetime="2020-07-09 12:00:00" /usr/local/mydata/log/mysql-bin.004735 | more
# 按位置范围查询，分页查看
/usr/local/mysql/bin/mysqlbinlog --no-defaults --database=mydb --start-position=188002427 --stop-position=258376950 /usr/local/mydata/log/mysql-bin.004735 | more

# 按时间范围查询，并将结果导出到新的文件中，方便查看
/usr/local/mysql/bin/mysqlbinlog --no-defaults --database=mydb --start-datetime="2020-07-09 10:00:00" --stop-datetime="2020-07-09 12:00:00" /usr/local/mydata/log/mysql-bin.004735 > /usr/local/mydata/binlog4375
# 按位置范围查询，并将结果导出到新的文件中，方便查看
/usr/local/mysql/bin/mysqlbinlog --no-defaults --database=mydb --start-position=188002427 --stop-position=258376950 /usr/local/mydata/log/mysql-bin.004735 > /usr/local/mydata/binlog4375
```

上述查询的结果无法直接阅读，如下：

```bash
/usr/local/mysql/bin/mysqlbinlog --no-defaults --database=mydb  --start-datetime="2020-07-09 10:00:00" --stop-datetime="2020-07-09 12:00:00" /usr/local/mydata/log/mysql-bin.004735 | more
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!40019 SET @@session.max_insert_delayed_threads=0*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#200709  8:53:46 server id 20001  end_log_pos 120 CRC32 0xaf81b3f4 	Start: binlog v 4, server v 5.6.42-log created 200709  8:53:46
BINLOG '
mmoGXw8hTgAAdAAAAHgAAAAAAAQANS42LjQyLWxvZwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAEzgNAAgAEgAEBAQEEgAAXAAEGggAAAAICAgCAAAACgoKGRkAAfSz
ga8=
'/*!*/;
# at 188002427
#200709 10:00:00 server id 20001  end_log_pos 188002501 CRC32 0xf0727217 	Query	thread_id=313330	exec_time=0	error_code=0
SET TIMESTAMP=1594260000/*!*/;
SET @@session.pseudo_thread_id=313330/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=2097152/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8 *//*!*/;
SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=192/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
BEGIN
/*!*/;
# at 188002501
#200709 10:00:00 server id 20001  end_log_pos 188002674 CRC32 0x50c4e1d3 	Table_map: `mydb`.`mytable` mapped to number 301
# at 188002674
#200709 10:00:00 server id 20001  end_log_pos 188002965 CRC32 0x4ad4cbb4 	Write_rows: table id 301 flags: STMT_END_F

BINLOG '
IHoGXxMhTgAArQAAAHKxNAsAAC0BAAAAAAEABm5iZ3JpZAASZ3JpZF9ldmVudF94dF90YXNrACgP
Dw8P/AMICA8PCA8PDw8PDw8PDw8PDw8PDw8PDw8PDw8IDw8PDw8DQ2AAYABgAB4AAmAA3AUeANwF
HgAsAR4AEgCWAJYAWgASAB4AEgBgAJYAYADcBRIABgCWAJYABgBgAGAAYAAeAGAAYAD+/////9Ph
xFA=
IHoGXx4hTgAAIwEAAJWyNAsAAC0BAAAAAAEAAgAo//////8Q+//wfCA4YTkzMzVjNzcyZmEzOGQz
MDE3MzMxNGQzYmJlNTRkNiA4YTkzMzVjNzcyZmEzOGQzMDE3MzMxNGQzYjUwNTRjYyA4YTkzMzVj
NzcyZmEzOGQzMDE3MzMxNGQzYjUwNTRjZQZ1bmRvbmUAAAAArjtNMXMBAACuO00xcwEAAAAAAAAA
AAAAC2V2ZW50SGFuZGxlIGJiNGE5ZGEzNWU5ZGFhYWEwMTVlOWRiZjdmZTgwMDRmEgDnjq/kv53m
o4Dmn6XmraPluLgCMzAgZmY4MDgwODE1ZDU1MTMyMDAxNWQ1ODljNzIzNTAwMTMAAAAAAAAAAAAA
AAC0y9RK
'/*!*/;
# at 188002965
#200709 10:00:00 server id 20001  end_log_pos 188002996 CRC32 0x8c0912cc 	Xid = 15280903421
COMMIT/*!*/;
# at 188002996
#200709 10:00:00 server id 20001  end_log_pos 188003070 CRC32 0xd595d036 	Query	thread_id=313330	exec_time=0	error_code=0
SET TIMESTAMP=1594260000/*!*/;
--More--
```

---

可在上面的基础上增加 `--base64-output=decode-rows -v` 参数将基于行的事件解码成一个SQL语句，如下：

```bash
/usr/local/mysql/bin/mysqlbinlog --no-defaults --database=mydb --base64-output=decode-rows -v --start-datetime="2020-07-09 10:00:00" --stop-datetime="2020-07-09 12:00:00" /usr/local/mydata/log/mysql-bin.004735 | more
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!40019 SET @@session.max_insert_delayed_threads=0*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#200709  8:53:46 server id 20001  end_log_pos 120 CRC32 0xaf81b3f4 	Start: binlog v 4, server v 5.6.42-log created 200709  8:53:46
# at 188002427
#200709 10:00:00 server id 20001  end_log_pos 188002501 CRC32 0xf0727217 	Query	thread_id=313330	exec_time=0	error_code=0
SET TIMESTAMP=1594260000/*!*/;
SET @@session.pseudo_thread_id=313330/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=2097152/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8 *//*!*/;
SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=192/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
BEGIN
/*!*/;
# at 188002501
#200709 10:00:00 server id 20001  end_log_pos 188002674 CRC32 0x50c4e1d3 	Table_map: `mydb`.`mytable` mapped to number 301
# at 188002674
#200709 10:00:00 server id 20001  end_log_pos 188002965 CRC32 0x4ad4cbb4 	Write_rows: table id 301 flags: STMT_END_F
### INSERT INTO `mydb`.`mytable`
### SET
###   @1='8a9335c772fa38d30173314d3bbe54d6'
###   @2='8a9335c772fa38d30173314d3b5054cc'
###   @3='8a9335c772fa38d30173314d3b5054ce'
###   @4='undone'
###   @5=NULL
###   @6=0
###   @7=1594260011950
###   @8=1594260011950
###   @9=NULL
###   @10=NULL
###   @11=0
###   @12=NULL
###   @13=NULL
###   @14=NULL
###   @15=NULL
###   @16=NULL
###   @17=NULL
###   @18=NULL
###   @19=NULL
###   @20=NULL
###   @21=NULL
###   @22=NULL
###   @23=NULL
###   @24=NULL
###   @25='eventHandle'
###   @26='bb4a9da35e9daaaa015e9dbf7fe8004f'
###   @27='环保检查正常'
###   @28='30'
###   @29=NULL
###   @30=NULL
###   @31=NULL
###   @32=NULL
###   @33='ff8080815d551320015d589c72350013'
###   @34=0
###   @35=NULL
###   @36=NULL
###   @37=NULL
###   @38=NULL
###   @39=NULL
###   @40=0
# at 188002965
#200709 10:00:00 server id 20001  end_log_pos 188002996 CRC32 0x8c0912cc 	Xid = 15280903421
COMMIT/*!*/;
# at 188002996
#200709 10:00:00 server id 20001  end_log_pos 188003070 CRC32 0xd595d036 	Query	thread_id=313330	exec_time=0	error_code=0
SET TIMESTAMP=1594260000/*!*/;
BEGIN
/*!*/;
# at 188003070
```

### 2.8 根据 binlog 日志恢复数据（使用 mysqlbinlog）

```bash
# 注意数据备份的时间节点、需要恢复数据的精确位置（起止），千万不要重复执行了相同的数据
/usr/local/mysql/bin/mysqlbinlog --no-defaults --database=mydb --start-position=188002427 --stop-position=258376950 /usr/local/mydata/log/mysql-bin.004735 | /usr/local/mysql/bin/mysql -uroot -p123456 -v mydb
```

## 3 数据恢复实战

### 3.1 准备测试数据

```sql
CREATE DATABASE mydb DEFAULT CHARACTER SET utf8mb4;

USE mydb;

CREATE TABLE `t_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `username` varchar(255) DEFAULT NULL COMMENT '用户名',
  `password` varchar(255) DEFAULT NULL COMMENT '密码',
  `enabled` tinyint(1) DEFAULT '1' COMMENT '是否启用：1 启用；0 未启用',
  `locked` tinyint(1) DEFAULT '0' COMMENT '是否锁定：1 锁定；0 未锁定',
  `address` varchar(255) DEFAULT NULL COMMENT '地址',
  `nick_name` varchar(255) DEFAULT NULL COMMENT '昵称',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

INSERT INTO t_user(username,password,address,create_time,update_time) VALUES ('user01','123456','HZ',NOW(),NOW());
INSERT INTO t_user(username,password,address,create_time,update_time) VALUES ('user02','123456','HZ',NOW(),NOW());
INSERT INTO t_user(username,password,address,create_time,update_time) VALUES ('user03','123456','HZ',NOW(),NOW());
INSERT INTO t_user(username,password,address,create_time,update_time) VALUES ('user04','123456','HZ',NOW(),NOW());
```

### 3.2 模拟误操作

假如在 `11:00` 做了如下误操作，导致部分数据错误：

```sql
INSERT INTO t_user(username,password,address,create_time,update_time) VALUES ('user05','123456','HZ',NOW(),NOW());
INSERT INTO t_user(username,password,address,create_time,update_time) VALUES ('user06','123456','HZ',NOW(),NOW());
INSERT INTO t_user(username,password,address,create_time,update_time) VALUES ('user07','123456','HZ',NOW(),NOW());
INSERT INTO t_user(username,password,address,create_time,update_time) VALUES ('user08','123456','HZ',NOW(),NOW());

DELETE FROM t_user WHERE id=1;
DELETE FROM t_user WHERE id=5;

UPDATE t_user SET address='SH' where id=2;
UPDATE t_user SET address='SH' where id=6;
```

### 3.3 模拟误操作之后数据的正常变更

```sql
INSERT INTO t_user(username,password,address,create_time,update_time) VALUES ('user09','123456','HZ',NOW(),NOW());
INSERT INTO t_user(username,password,address,create_time,update_time) VALUES ('user10','123456','HZ',NOW(),NOW());

DELETE FROM t_user WHERE id=3;
DELETE FROM t_user WHERE id=7;

UPDATE t_user SET address='BJ' where id=2;
UPDATE t_user SET address='BJ' where id=4;
UPDATE t_user SET address='BJ' where id=8;
```

### 3.4 根据 binlog 日志恢复数据

> 前提：假设在 `10:00` 做了数据备份（云服务器自动备份或手动备份，如果一直没备份，那没的玩）。

1. 通过备份将数据库的数据还原到 `10:00` 的状态，之后在这基础上通过 binlog 日志将 3.2 中做的误操作跳过。
2. 通过备份的时间节点（`10:00`）查询 binlog 日志（比如：`mysql-bin.000002`），找到对应的位置 position（比如分别为：`63926791`）。
3. 通过 3.2 中大致的时间节点（比如：`11:00-11:05`）查询 binlog 日志（比如：`mysql-bin.000002`），找到对应的开始位置和结束位置 position（比如分别为：`63933358` 和 `63938729`）。
4. 开始还原 `10:00-11:00` 的数据： `/usr/local/mysql/bin/mysqlbinlog --no-defaults --database=mydb --start-position=63926791 --stop-position=63933358 /usr/local/mydata/log/mysql-bin.000002 | /usr/local/mysql/bin/mysql -uroot -p123456 -v mydb`。
5. 开始还原 `11:05` 之后的数据： `/usr/local/mysql/bin/mysqlbinlog --no-defaults --database=mydb --start-position=63938729 /usr/local/mydata/log/mysql-bin.000002 | /usr/local/mysql/bin/mysql -uroot -p123456 -v mydb`。

另外注意特殊情况：涉及到多个 binlog 日志文件。

---

- [MySQL 教程合集](https://mp.weixin.qq.com/s/jflrWU62pBtevS62lEIHkQ)（微信左下方**阅读全文**可直达）。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)