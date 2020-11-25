---
title: MySQL 主从部署
date: 2018-09-12 16:42:36
categories: MySQL
tags: [MySQL, 主从]
toc: true
---
认识 MySQL 主从，并手把手带你实现 MySQL 主从部署，图文并茂。
<!-- more -->

## 1 概述

### 1.1 什么是数据库主从

主库把所有的操作都记入二进制日志，从库通过网络把主库日志拷贝入自己的日志，从库读取日志进行数据更改。

示意图：

![](https://static.oschina.net/uploads/space/2018/0227/144604_An7d_593078.png)

原理图：

![](https://oscimg.oschina.net/oscnet/5b2cda22da660e17a3f24559d09267df8fc.jpg)

![](https://oscimg.oschina.net/oscnet/8ef3ddf354a9fcf371de2544bd66f8e8d82.jpg)

**注意事项：**

- 主数据库和从数据库版本应一致，如果不一致，从数据库版本应高于主数据库版本。
- 主从同步实质是同步数据库操作，不是保证两者数据一致。所以启动主从前，应先保证两者数据一致。
- 从库的数据相对主库有滞后性。
- 主从配置会影响主库的性能，从库越多对主库的影响越大。
- 当一次配置成功后，主从随着数据库启动而启动。 master 随主库 mysql 启动而启动， slave 随从库 mysql 启动而启动。
- 必须先启动主库，再启动从库。
- 只要主库正常，从库停机后启动，主从自动可以把主库数据同步过来。

### 1.2 主从的同步方式

- **同步复制**：所谓的同步复制，意思是 master 的变化，必须等待 slave-1, slave-2,..., slave-n 完成后才能返回。这样，显然不可取，也不是 MySQL 复制的默认设置。比如，在 WEB 前端页面上，用户增加了条记录，需要等待很长时间。
- **异步复制**：如同 AJAX 请求一样。master 只需要完成自己的数据库操作即可。至于 slaves 是否收到二进制日志，是否完成操作，不用关心。MySQL 的默认设置。
- **半同步复制**：master 只保证 slaves 中的一个操作成功，就返回，其他 slave 不管。

### 1.3 主从有什么用

- **读写分离**：把统计等费时、占资源高的操作移到从库。
- **异地备份**：类似于高可用的功能，一旦 master 挂了，可以让 slave 顶上去，同时 slave 提升为 master 。

### 1.4 主从架构选择

数据库主从架构有多种选择，每种架构都有其适应性，我认为最有用的有如下两种：

#### 1.4.1 一主多从架构

![](https://oscimg.oschina.net/oscnet/dd449a7fa8eb8fa3b754b230c0cfd996e05.jpg)

当写入操作较少，读操作较多可以采取此架构。可以有效的均衡压力，但是，当 slave 增加到一定数量时， slave 对 master 的负载以及网络带宽都会成为一个严重的问题。这种结构虽然简单，但是，它却非常灵活，足够满足大多数应用需求。一些建议：

- 不同的 slave 扮演不同的作用（例如使用不同的索引，或者不同的存储引擎）。
- 用一个 slave 作为备用 master ，只进行复制。
- 用一个远程的 slave ，用于灾难恢复。

#### 1.4.2 级联复制架构

![](https://static.oschina.net/uploads/space/2018/0227/144735_PQb1_593078.png)

![](https://oscimg.oschina.net/oscnet/66cf6685301a4a5233b86dada155b937b9d.jpg)

在有些应用场景中，可能读写压力差别比较大，读压力特别的大，一个 Master 可能需要上10台甚至更多的 Slave 才能够支撑注读的压力。这时候， Master 就会比较吃力了，因为仅仅连上来的 Slave IO 线程就比较多了，这样写的压力稍微大一点的时候， Master 端因为复制就会消耗较多的资源，很容易造成复制的延时。

## 2 主库 master 配置

### 2.1 创建用户并授权

```bash
# 登录 mysql
/usr/local/mysql/bin/mysql -uroot -p123456 -S /usr/local/mydata/tmp/mysql.sock

mysql> CREATE USER 'repl'@'%' IDENTIFIED BY '123456';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
mysql> flush privileges;
```

### 2.2 修改配置文件

```bash
vi /etc/my.cnf
# 在 [mysqld] 中增加
server-id=1
log-bin=/usr/local/mydata/log/mysql-bin.log
binlog_format=mixed
expire_logs_days=7

# 也可不指定，代表默认所有
binlog-do-db=db1
binlog-ignore-db=mysql
binlog-ignore-db=information_schema
binlog-ignore-db=performance_schema
binlog-ignore-db=test

# 参数说明：
# server-id：唯一ID，主库建议为1，必须。
# log-bin：是否开启二进制日志，必须。
# binlog_format：日志记录格式，推荐。
# expire_logs_days：日志过期删除的时间。
# binlog-do-db：需要同步的数据库，多个时用多行。
# binlog-ignore-db：不需要同步的数据库，多个时用多行。
```

### 2.3 启动 mysql ，会自动启动 master

```bash
# 启动 mysql
/bin/bash /usr/local/mysql/bin/mysqld_safe --defaults-file=/usr/local/mydata/etc/my.cnf &
```

### 2.4 查看主库状态

```bash
# 登录 mysql
/usr/local/mysql/bin/mysql -uroot -p123456 -S /usr/local/mydata/tmp/mysql.sock

mysql> show master status \G;
*************************** 1. row ***************************
            File: mysql-bin.001167
        Position: 439884600
    Binlog_Do_DB: db1
Binlog_Ignore_DB: mysql,information_schema,performance_schema,test
1 row in set (0.00 sec)
```

## 3 从库 slave 配置

### 3.1 修改配置文件

```bash
vi /etc/my.cnf
# 在 [mysqld] 中增加
server-id=2
# log-bin=/usr/local/mydata/log/mysql-bin.log
# log_slave_updates=1
relay-log=/usr/local/mydata/log/mysql-relay-bin.log
read_only=1
expire_logs_days=7

# 也可不指定，代表默认所有
replicate-do-db=db1
replicate-ignore-db=mysql
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema
replicate-ignore-db=test

# 参数说明：
# server-id：唯一ID，必须。
# log-bin：是否开启二进制日志，单纯作为从库不需要配置，如果该从库同时作为其他从库的主库时则必须设置。
# log_slave_updates：将复制事件写进自己的二进制日志，单纯作为从库不需要配置，如果该从库同时作为其他从库的主库时则必须设置。
# relay-log：中继日志。
# read_only 只读，但对root用户无效。
# expire_logs_days：日志过期删除的时间。
# replicate-do-db：需要同步的数据库，多个时用多行。
# replicate-ignore-db：不需要同步的数据库，多个时用多行。
```

### 3.2 启动 mysql ，会自动启动 slave

```bash
# 启动 mysql
# --read_only=1  # 打开只读，从库设置
/bin/bash /usr/local/mysql/bin/mysqld_safe --defaults-file=/usr/local/mydata/etc/my.cnf --read_only=1 &

# 登录 mysql
/usr/local/mysql/bin/mysql -uroot -p123456 -S /usr/local/mydata/tmp/mysql.sock

# 第1次需要配置主从
mysql> stop slave;
mysql> change master to master_host='192.168.71.57',
	master_user='repl',
	master_password='123456',
	master_log_file='mysql-bin.001167',
	master_log_pos=439884600,
	master_port=3306;
mysql> start slave;

# 停止主从复制
# mysql> stop slave;

# 清除从库的同步复制信息，包括连接信息和二进制文件名、位置（使用 show slave status 将不会有输出）
# mysql> reset slave all;
```

### 3.3 查看从库状态

```bash
# 登录 mysql
/usr/local/mysql/bin/mysql -uroot -p123456 -S /usr/local/mydata/tmp/mysql.sock

mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.71.57  # 主服务器地址。
                  Master_User: repl  # 授权用户名，尽量避免使用root。
                  Master_Port: 3306  # 数据库端口，部分版本没有此行。
                Connect_Retry: 60
              Master_Log_File: mysql-bin.001167  # 主库最大的二进制日志文件。
          Read_Master_Log_Pos: 647719681  # 从库读取主库二进制日志文件，已经读取到的位置，>=Exec_Master_Log_Pos。
               Relay_Log_File: mysql-relay-bin.002932
                Relay_Log_Pos: 647719828
        Relay_Master_Log_File: mysql-bin.001167  # 从库已经收到的主库最大的二进制日志文件。
             Slave_IO_Running: Yes  # 此状态必须YES。
            Slave_SQL_Running: Yes  # 此状态必须YES。
              Replicate_Do_DB: db1
          Replicate_Ignore_DB: mysql,information_schema,performance_schema,test
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0  # 错误编号
                   Last_Error:    # 错误描述
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 647719681  # 从库执行主库二进制日志文件，已经执行到的位置。
              Relay_Log_Space: 647720028
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0  # 从库落后主库的秒数，大部分情况是准的。
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
1 row in set (0.01 sec)
```

## 4 问题汇总

### 4.1 跨库更新问题

举例：某两个数据库已经实现了主从同步，现在主库中有两个数据库 test01 和 test02 ，然后 test01 中有一张表 table01 ，如果在 my.cnf 的参数里面设置了 replicate_do_db=test01 , test02 ，即只同步这两个库的数据，然后执行以下的更新语句：

```bash
use test01;  
update test01.table1 set......  
```

执行的结果是主从库都能看到更新的数据。但如果是另外一种执行的情况进行更新语句：

```bash
use test02;  
update test01.table1 set......  
```

执行的结果是主库能够看到数据，但是从库却无法看到更新的数据。

原因：设置 replicate_do_db 后， MySQL 执行 sql 前检查的是当前默认数据库，所以跨库更新语句在 Slave 上会被忽略。而对于跨库更新 SQL 语句的问题， replicate_wild_do_table 可以解决，即在 my.cnf 的参数里面设置

```bash
# （正确写法）  
replicate_wild_do_table=test01.%  
replicate_wild_do_table=test02.%  
# （错误写法）  
replicate_wild_do_table=test01.%,test02.%
```

注意需要同步的库必须分行写而不能在同一行用逗号隔开，否则在同步的时候该参数不生效。

**建议：同步所有数据，如果确实不需要同步某几个库，一定要确认没有跨库问题。**

### 4.2 从库同步错误问题

- 跳过 1 个错误

```bash
# 登入从库查看错误类型
show slave status \G;
# 停止从库
slave stop;
# 跳过一个错误
SET GLOBAL SQL_SLAVE_SKIP_COUNTER=1;
# 启动从库
slave start;
# 稍后再次查看从库是否正常（ Slave_IQ_Running 和 Slave_SQL_Running 均为 YES 状态表示正常）。
```

- 跳过 1 类错误

```bash
vi /etc/my.cnf  
# 在 [mysqld] 中增加

# 跳过指定类型的错误，Last_SQL_Errno 可查看错误编号  
# slave-skip-errors=1062,1053,1146

# 跳过所有错误
# slave-skip-errors=all
```

---

- [MySQL 教程合集](https://mp.weixin.qq.com/s/jflrWU62pBtevS62lEIHkQ)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)