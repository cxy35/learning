---
title: MySQL 常见问题
date: 2018-11-06 15:35:24
categories: MySQL
tags: [MySQL, 问题]
toc: true
---
本文记录 MySQL 安装、启动、运行过程中出现的问题汇总，持续更新中。
<!-- more -->

## 1 Plugin 'InnoDB' registration as a STORAGE ENGINE failed

原因：启动时报错，配置文件修改导致出问题。

解决：停服务，删除那些日志文件，再启动服务。

```bash
rm -f /usr/local/mysql/data/ib_logfile*

# 下面的可能也需要删除
rm -f /usr/local/mysql/data/mysql-bin*
```

## 2 ERROR 1418 (HY000): This function has none of DETERMINISTIC, NO SQL, or READS SQL DATA in its declaration and binary logging is enabled (you *might* want to use the less safe log_bin_trust_function_creators variable)

解决：在 my.cnf 配置文件中添加：log_bin_trust_function_creators=1

## 3 Waiting for table flush

最近遇到一个案例，很多查询被阻塞没有返回结果，使用 show processlist 查看，发现不少 MySQL 线程处于 Waiting for table flush 状态，查询语句一直被阻塞，只能通过 Kill 进程来解决。

原因：

有些时候，是由于 **`lock table t_test read`** 引起的阻塞。但生产环境中，很多时候可能是由于 **慢查询** 导致 flush table 一直无法关闭该表而一直处于等待状态。

另外，网上有个案例，mysqldump 备份时，如果没有使用参数 —single-transaction 或由于同时使用了flush-logs 与 —single-transaction 两个参数也可能引起这样的等待场景，这个两个参数放在一起，会在开始 dump 数据之前先执行一个 FLUSH TABLES 操作。

解决：

出现 Waiting for table flush 时，我们一般需要找到那些表被 lock 住或那些慢查询导致 flush table 一直在等待而无法关闭该表。然后 Kill 掉对应的线程即可，但是如何精准定位是一个挑战，尤其是生产环境，你使用 show processlist 会看到大量的线程。让你眼花缭乱的，怎么一下子定位问题呢？

- 对于慢查询引起的其它线程处于 Waiting for table flush 状态的情形：

可以查看 show processlist 中 Time 值很大的线程，然后甄别确认后 Kill 掉。有种规律就是这个线程的 Time 列值必定比被阻塞的线程要高，这个就能过滤很多记录。

- 对于 lock table t_test read 引起的其它线程处于 Waiting for table flush 状态的情形：

对于 lock table t_test read 这种情况，这种会话可能处于 Sleep 状态，而且它也不会出现在 show engine innodb status \G 命令的输出信息中。 即使 show open tables where in_use >=1; 能找到是那张表被 lock 住了，但是无法定位到具体的线程（连接），其实这个是一个头痛的问题，可以使用 **MySQL监控利器-Innotop** 。

另外，在官方文档中 **`ALTER TABLE, RENAME TABLE, REPAIR TABLE, ANALYZE TABLE, OPTIMIZE TABLE`** 都能引起这类等待。

## 4 Host 'xxx' is blocked because of many connection errors; unblock with 'mysqladmin flush-hosts'

原因：

同一个 ip 在短时间内产生太多（超过 MySQL 数据库 max_connect_errors 的最大值）中断的数据库连接而导致的阻塞。

说明 MySQL 已经得到了大量( max_connect_errors )的主机 'hostname' 的在中途被中断了的连接请求。在 max_connect_errors 次失败请求后， MySQL 认定出错了(像来自一个黑客的攻击)，并且阻止该站点进一步的连接，直到某人执行命令 mysqladmin flush-hosts 。

解决：

1. 提高允许的 max_connect_errors 数量（治标不治本）：

    1. 进入 MySQL 数据库查看 max_connect_errors： show variables like '%max_connect_errors%';
    2. 修改 max_connect_errors 的数量为 1000： set global max_connect_errors = 1000;
    3. 查看是否修改成功：show variables like '%max_connect_errors%';

2. 使用 mysqladmin flush-hosts 命令清理一下 hosts 文件（不知道 mysqladmin 在哪个目录下可以使用命令查找： whereis mysqladmin ）

在查找到的目录下使用命令修改：/usr/local/mysql/bin/mysqladmin flush-hosts -h10.19.11.33 -P3306 -uroot -p123456

备注：

- 其中端口号，用户名，密码都可以根据需要来添加和修改。
- 配置有 master/slave 主从数据库的要把主库和从库都修改一遍的，如果连了多个数据库，则都需要处理。
- 第 2 步也可以在数据库中进行，命令如下： flush hosts;

---

- [MySQL 教程合集](https://mp.weixin.qq.com/s/jflrWU62pBtevS62lEIHkQ)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)