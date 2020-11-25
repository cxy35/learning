---
title: MySQL 备份与恢复 - mydumper
date: 2018-10-13 15:42:59
categories: MySQL
tags: [MySQL, 备份, mydumper]
toc: true
---
mydumper 是第三方支持 mysql 的备份恢复工具，支持多线程，备份与恢复速度快。
<!-- more -->

## 1 准备工作

### mydumper 特性

1. 支持多线程导出数据，速度比 mysqldump 快。
2. 支持一致性备份，使用 FTWRL(FLUSH TABLES WITH READ LOCK) 会阻塞 DML 语句,保证备份数据的一致性。
3. 支持将导出文件压缩，节约空间。
4. 支持多线程恢复。
5. 支持以守护进程模式工作，定时快照和连续二进制日志。
6. 支持按照指定大小将备份文件切割。
7. 数据与建表语句分离。

### mydumper 主要工作步骤

1. 主线程 FLUSH TABLES WITH READ LOCK, 施加全局只读锁，以阻止 DML 语句写入，保证数据的一致性。
2. 读取当前时间点的二进制日志文件名和日志写入的位置并记录在 metadata 文件中，以供即时点恢复使用。
3. START TRANSACTION WITH CONSISTENT SNAPSHOT; 开启读一致事务。
4. 启用 N 个（线程数可以指定，默认是4） dump 线程导出表和表结构 。
5. 备份非事务类型的表。
6. 主线程 UNLOCK TABLES，备份完成非事务类型的表之后，释放全局只读锁。
7. dump InnoDB tables, 基于事物导出 InnoDB 表。
8. 事物结束。

### 下载安装包

mydumper 版本：mydumper-0.9.1

- 手动下载地址：[https://github.com/maxbube/mydumper/releases](https://github.com/maxbube/mydumper/releases)
- linux 自动获取：wget https://github.com/maxbube/mydumper/archive/v0.9.1.tar.gz

## 2 安装

```bash
# 解压，重命名
tar -xvzf mydumper-0.9.1.tar.gz -C /usr/local/
mv /usr/local/mydumper-0.9.1 /usr/local/mydumper

# 查看注意事项
#cat /usr/local/mydumper/README

# 安装依赖
yum -y install glib2-devel mysql-devel zlib-devel pcre-devel zlib gcc-c++ gcc cmake

# 编译安装，完成之后在会 /usr/local/mydumper 目录下生成 mydumper 和 myloader 二进制文件
cd /usr/local/mydumper
cmake .
make && make install

# 编译报错的话查看报错信息，一般是因为依赖包的问题
# cat /usr/local/mydumper/CMakeFiles/CMakeOutput.log
```

具体的编译安装过程如下：

```bash
[root@localhost local]$ cd /usr/local/mydumper
[root@localhost mydumper]$ cmake .
-- The C compiler identification is GNU 4.4.7
-- The CXX compiler identification is GNU 4.4.7
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Using mysql-config: /usr/bin/mysql_config
-- Found MySQL: /usr/include/mysql, /usr/lib64/mysql/libmysqlclient_r.so;/usr/lib64/libz.so;/usr/lib64/libpthread.so;/usr/lib64/libcrypt.so;/usr/lib64/libnsl.so;/usr/lib64/libm.so;/usr/lib64/libpthread.so;/usr/lib64/libssl.so;/usr/lib64/libcrypto.so
-- Found ZLIB: /usr/lib64/libz.so (found version "1.2.3") 
-- Found PkgConfig: /usr/bin/pkg-config (found version "0.23") 
-- checking for one of the modules 'glib-2.0'
-- checking for one of the modules 'gthread-2.0'
-- checking for module 'libpcre'
--   found libpcre, version 7.8
-- Found PCRE: /usr/include  

CMake Warning at docs/CMakeLists.txt:9 (message):
  Unable to find Sphinx documentation generator


-- ------------------------------------------------
-- MYSQL_CONFIG = /usr/bin/mysql_config
-- CMAKE_INSTALL_PREFIX = /usr/local
-- BUILD_DOCS = ON
-- WITH_BINLOG = OFF
-- RUN_CPPCHECK = OFF
-- Change a values with: cmake -D<Variable>=<Value>
-- ------------------------------------------------
-- 
-- Configuring done
-- Generating done
-- Build files have been written to: /usr/local/mydumper
[root@localhost mydumper]$ make && make install
Scanning dependencies of target mydumper
[ 25%] Building C object CMakeFiles/mydumper.dir/mydumper.c.o
[ 50%] Building C object CMakeFiles/mydumper.dir/server_detect.c.o
[ 75%] Building C object CMakeFiles/mydumper.dir/g_unix_signal.c.o
Linking C executable mydumper
[ 75%] Built target mydumper
Scanning dependencies of target myloader
[100%] Building C object CMakeFiles/myloader.dir/myloader.c.o
Linking C executable myloader
[100%] Built target myloader
[ 75%] Built target mydumper
[100%] Built target myloader
Install the project...
-- Install configuration: ""
-- Installing: /usr/local/bin/mydumper
-- Removed runtime path from "/usr/local/bin/mydumper"
-- Installing: /usr/local/bin/myloader
-- Removed runtime path from "/usr/local/bin/myloader"
[root@localhost mydumper]$ 
```

## 3 使用

```bash
# 导出整个库
/usr/local/bin/mydumper -u root -S /srv/my3308/run/mysql.sock -B trade_platform -o /data/trade_platform

# 只导出表结构，不导数据
/usr/local/bin/mydumper -u root -S /srv/my3308/run/mysql.sock -B trade_platform -d -o /data/trade_platform

# 以压缩的方式导出的文件
/usr/local/bin/mydumper -u root -S /srv/my3308/run/mysql.sock -B trade_platform -c -o /data/trade_platform

# 备份文件以 .gz 的格式压缩
# ls
metadata trade_platform.config.sql.gz trade_platform.trade_order-schema.sql.gz
trade_platform.config-schema.sql.gz trade_platform-schema-create.sql.gz trade_platform.trade_order.sql.gz

# 使用正则表达式。其中正则表达式可以是 --regex=order.*  导出所有 order 开头的表
/usr/local/bin/mydumper -u root -S /srv/my3308/run/mysql.sock --regex='^(?!(mysql|test))' -o /data/bk20170120
```

mydumper 导出的文件包括 `metadata trade\_platform.config.sql trade\_platform.order.sql` ：
- metadata：记录导出时binlog的位点信息，如果启用gtid ，则记录gtid信息。

    ```bash
    Started dump at: 2017-01-20 17:26:53  
    SHOW MASTER STATUS:  
      Log: mysql-bin.000025  
      Pos: 505819083  
      GTID:  
    Finished dump at: 2017-01-20 17:27:02  
    ```

- db.table.sql        :数据文件，insert语句  
- db.table-schema.sql :包含建表语句  
- db-schema.sql       :包含建库语句

相比 mysqldump ， mydumper 导出的文件形式是每个表一个文件，对于开发/测试环境的误操作恢复十分有效。

**实际使用：**

```bash
# 导出数据库 mydb
/usr/local/bin/mydumper -u root -h 127.0.0.1 -P 3306 -S /usr/local/griddata/tmp/mysql.sock --skip-tz-utc --less-locking -B mydb -o /usr/local/mydata/mydb_0708 -v 3 | tee -a /tmp/mydb_0708.log

# ** Message: Thread 2 dumping table for `mydb`.`t_user`
# ** Message: Thread 2 dumping schema for `mydb`.`t_user`
# ** Message: Thread 2 dumping view for `mydb`.`view_user`
# ** Message: Thread 2 shutting down
# ** Message: Thread 4 shutting down
# ** Message: Thread 1 shutting down
# ** Message: Thread 3 shutting down
# ** Message: Finished dump at: 2019-07-08 22:32:30

[root@localhost mydata]$ cat mydb_0708/metadata 
Started dump at: 2019-07-08 16:41:24
SHOW MASTER STATUS:
	Log: mysql-bin.000739
	Pos: 71397008
	GTID:

SHOW SLAVE STATUS:
	Host: 10.68.128.149
	Log: mysql-bin.000763
	Pos: 77168544
	GTID:

Finished dump at: 2019-07-08 22:32:30

# 导入数据库 mydb
/usr/local/bin/myloader -u root -h 127.0.0.1 -P 3308 -o -e -B mydb -d /usr/local/mydata/mydb_0708 -t 8 -v 3 | tee -a /tmp/mydb_0710.log

# ** Message: Dropping table or view (if exists) `mydb`.`t_user`
# ** Message: Creating table `mydb`.`t_user`
# ** Message: Thread 3 restoring `mydb`.`t_user` part 0
```

## 4 参数说明

### 4.1 mydumper 常用参数

```bash
-B, --database 要导出的dbname
-T, --tables-list 需要导出的表名,导出多个表需要逗号分隔，t1[,t2,t3 ....] 
-o, --outputdir 导出数据文件存放的目录，mydumper会自动创建
-s, --statement-size 生成插入语句的字节数, 默认1000000字节
-r, --rows Try to split tables into chunks of this many rows. This option turns off --chunk-filesize
-F, --chunk-filesize 切割表文件的大小，默认单位是 MB ，如果表大于
-c, --compress 压缩导出的文件
-e, --build-empty-files 即使是空表也为表创建文件
-x, --regex 使用正则表达式匹配 db.table 
-i, --ignore-engines 忽略的存储引擎，多个值使用逗号分隔
-m, --no-schemas 只导出数据，不导出建库建表语句
-d, --no-data 仅仅导出建表结构，创建db的语句
-G, --triggers 导出触发器
-E, --events 导出events
-R, --routines 导出存储过程和函数
-k, --no-locks 不执行临时的只读锁，会导致备份不一致 。WARNING: This will cause inconsistent backups
--less-locking 最小化在innodb表上的锁表时间 --butai
-l, --long-query-guard 设置长时间执行的sql 的时间标准
-K, --kill-long-queries 将长时间执行的sql kill
-D, --daemon 以守护进程的方式执行
-I, --snapshot-interval 创建导出快照的时间间隔，默认是 60s ，该参数只有在守护进程执行的时候有用。
-L, --logfile 指定mydumper输出的日志文件，默认使用控制台输出。
--tz-utc SET TIME_ZONE='+00:00' at top of dump to allow dumping of TIMESTAMP data when a server has data in different time zones or data is being moved between servers with different time zones, defaults to on use --skip-tz-utc to disable.
--skip-tz-utc
--use-savepoints 使用savepoints 减少MDL 锁事件 需要 SUPER 权限
--success-on-1146 Not increment error count and Warning instead of Critical in case of table doesn
```

### 4.2 myloader 常用参数

```bash
-d, --directory 备份文件的文件夹
-q, --queries-per-transaction 每次事物执行的查询数量，默认是1000
-o, --overwrite-tables 如果要恢复的表存在，则先drop掉该表，使用该参数，需要备份时候要备份表结构
-B, --database 需要还原的数据库
-e, --enable-binlog 启用还原数据的二进制日志
-h, --host The host to connect to
-u, --user Username with privileges to run the dump
-p, --password User password
-P, --port TCP/IP port to connect to
-S, --socket UNIX domain socket file to use for connection
-t, --threads 还原所使用的线程数，默认是4
-C, --compress-protocol 压缩协议
-V, --version 显示版本
-v, --verbose 输出模式, 0 = silent, 1 = errors, 2 = warnings, 3 = info, 默认为2
```

---

- [MySQL 教程合集](https://mp.weixin.qq.com/s/jflrWU62pBtevS62lEIHkQ)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)