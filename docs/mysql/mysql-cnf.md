---
title: MySQL 参数配置优化
date: 2018-09-26 09:25:47
categories: MySQL
tags: [MySQL, 优化]
toc: true
---
MySQL 参数配置优化，提升 MySQL 效率与稳定性。
<!-- more -->

## character_set_server

可设置成 utf8。

## default_character_set

可设置成 utf8。

## collation_server

可设置成 utf8_unicode_ci。

## init_connect

可设置成 'SET NAMES utf8' 。每个连接都会先执行 init_connect，进行连接的初始化。当一个连接进来时做一些操作，比如 'SET NAMES utf8' ，比如设置 autocommit 为 0 ，比如记录当前连接的ip来源和用户等信息到一个新表里，当做登陆日志信息。只有超级账户才可以设置( super_priv 权限)，超级账户无视 init_connect 设置(即 init_connect 的设置对来自超级账户的连接不生效)。

## log_error

可设置成 /usr/local/mydata/log/alert.log 。

## log_bin

可设置成 /usr/local/mydata/log/mysql_bin.log 。

## relay_log

可设置成 /usr/local/mydata/log/mysql_relay_bin.log 。

## slow_query_log_file

可设置成 /usr/local/mydata/log/mysql_slow.log 。

## read_only

可设置成 1 。

## max_connections

MySQL 服务端允许的最大连接会话数量。根据机器的配置来设置，如 CPU32 核，内存 64G ，可设置成 4000。

## max_user_connections

单个用户允许连接的最大会话数量，可设置成 4000。

## max_connect_errors

可设置成 90000000 。对于同一主机， 如果有超出该参数值个数的中断错误连接， 则该主机将被禁止连接。 如需对该主机进行解禁， 执行： FLUSH HOST 。

## wait_timeout、interactive_timeout

默认 28800 秒（即 8 小时），建议设置成 120 秒。注意数据库连接池的配置，避免取出的连接是已经被数据库清理掉的。

控制连接的最大空闲时间（可通过 show processlist 输出中 Sleep 状态的时间）的是 wait_timeout 参数（对于非交互式连接，类似于 jdbc 连接， wait_timeout=wait_timeout 。对于交互式连接，类似于 mysql 客户端连接， wait_timeout=interactive_timeout ）。

## lower_case_table_names

Windows 中默认 1 ， Linux 中默认 0 。表名忽略大小写。1=忽略，0=不忽略。可设置成1。设置1，大小写不敏感。创建的表，数据库都是以小写形式存放在磁盘上，对于sql语句都是转换为小写对表和DB进行查找。当想设置 lower_case_table_names = 1 时，在重启数据库实例之前就需要将原来的数据库和表转换为小写，否则将找不到。

## binlog_format

可设置成 MIXED 。

## expire_logs_days

可设置成 7 。

## master_info_repository

可设置成 TABLE 。

## max_binlog_size

可设置成 1G 。

## slow_query_log

可设置成 1 。

## long_query_time

可设置成 10 。

## innodb_file_per_table

建议设置成 ON ，一个表对应一个数据文件，否则所有表的数据都在 ibdata1 一个文件中。

## innodb_buffer_pool_size

默认 134217728 字节（即 128M ），建议设置成机器内存的 50%-80% ，如机器 64G 内存，则可设置成 45G 。这个参数控制 Innodb 本身的缓存大小，影响多少数据能在缓存中。

## innodb_lock_wait_timeout

可设置成 120 。事务锁等待超时时间。如果批量处理数据，有大事务时可临时调大。

## sort_buffer_size

建议设置成 1M 或 2M ，设置的值过大会造成系统内存不足。

我们一般可以通过增大 sort buffer 的大小来提高 order by 或者 group by 的处理性能。

Sort_Buffer_Size 是一个 connection 级参数，在每个 connection 第一次需要使用这个 buffer 的时候，一次性分配设置的内存。

Sort_Buffer_Size 并不是越大越好，由于是 connection 级的参数，过大的设置+高并发可能会耗尽系统内存资源。

文档说 “On Linux, there are thresholds of 256KB and 2MB where larger values may significantly slow down memory allocation” 。

## join_buffer_size

建议设置成 1M 或 2M ，设置的值过大会造成系统内存不足。

## max_allowed_packet

可设置成 16M 。

## back_log

可设置成 5000 。 MySQL 在瞬时能够接收的连接数，高并发时需要配置。

back_log 值指出在 MySQL 暂时停止回答新请求之前的短时间内多少个请求可以被存在堆栈中。也就是说，如果 MySQL 的连接数达到 max_connections 时，新来的请求将会被存在堆栈中，以等待某一连接释放资源，该堆栈的数量即 back_log ，如果等待连接的数量超过 back_log ，将不被授予连接资源。

## sync_binlog

可设置成 100 。

## innodb_flush_log_at_trx_commit

可设置成 2 。设为 1 当然是最安全的，但性能也是最差的（相对其他两个参数而言， 但不是不能接受） 。如果对数据一致性和完整性要求不高，完全可以设为 2 ，如果只追求性能，例如高并发写的日志服务器，设为 0 来获得更高性能。

## innodb_write_io_threads

## innodb_read_io_threads

## innodb_thread_concurrency

MySQL Innodb 并发涉及参数：[https://www.cnblogs.com/xinysu/p/6439715.html](https://www.cnblogs.com/xinysu/p/6439715.html)

这个是 innodb 内核的并发线程处理参数，即同一时刻能够进入 innodb 层次并发执行的线程数（**注意是并发不是并行**）。比如前端有 100 个连接，发来 1000 个 sql ，如果这个参数被设置成 2 。那么这 1000 个 sql 中，最多只有 2 个 sql 在 innodb 内核运行。其它都得等。(事实上，处理过程很复杂，可以先这么理解，不是所有 sql 都需要放在 Innodb 内核处理的)。

默认 0 ，则表示没有并发线程数限制，所有请求都会直接请求线程执行。注意：当 innodb_thread_concurrency 设置为 0 时，则 innodb_thread_sleep_delay 的设置将会被忽略，不起作用。如果数据库没出现性能问题时，使用默认值即可。

当 >0 ，则表示有并发数限制，当一个新的请求发起时，会检查当前并发线程数是否达到了 innodb_thread_concurrency 的限制值，如果有，则需要 sleep 一段时间（ sleep 的设置详见下一部分），然后再再次请求，如果再次请求时，当前并发数还是达到限制值，那么就会进入 FIFO 队列等待执行。当进入到内核执行时，会得到一个消费凭证 ticket ，则这个线程，在后面的多次进入 innodb 执行操作是都不需要重复上面的检查步骤，当把次数消费完，那么这个线程就会被驱逐，等待下次再次进入 Innodb ，再重新分配 ticket 。那些等待获取锁的线程则不会被计入到并发执行线程 innodb_thread_concurrency 的数量中。

**建议配置（来自官网）：**

- 当并发用户线程数量小于 64 ，建议设置 innodb_thread_concurrency=0 ；
- 如果负载不稳定，时而低，时而高到峰值，建议先设置 innodb_thread_concurrency=128 ，并通过不断的降低这个参数， 96, 80, 64 等等，直到发现能够提供最佳性能的线程数，例如，假设系统通常有 40 到 50 个用户，但定期的数量增加至 60，70 ，甚至 200 。你会发现，性能在 80 个并发用户设置时表现稳定，如果高于这个数，性能反而下降。在这种情况下，建议设置 innodb_thread_concurrency 参数为 80 ，以避免影响性能；
- 如果 DB 服务器上还允许其他应用，需要限制 MySQL 的线程使用情况，则可以设置可分配给 DB 的线程数，但是不建议 DB 上跑其他应用，也不建议这么设置，因为这样可能导致数据库没有对硬件最优使用；
- 设置过高值，可能会因为系统资源内部争夺导致性能下降；
- **在大多数情况下，最佳的值是小于并接近虚拟 CPU 的个数；**
- 定期监控和分析 DB ，因为随着数据库负载的变化，业务的增加， innodb_thread_concurrency 也需要动态的调整。

## innodb_thread_sleep_delay

5.6.3 版本前，需要反复测试才能确定 innodb_thread_sleep_delay 值，并且固定为一个值，在 5.6.3 版本后，因为 Innodb 自动调整 innodb_thread_sleep_delay 参数： Innodb_adaptive_max_sleep_delay ：最大 sleep 的时间，微秒为单位。可以通过设置参数 innodb_adaptive_max_sleep_delay 来限制 innodb_thread_sleep_delay 的最大值，不设置 innodb_thread_sleep_delay 的取值情况，让 Innodb 自动跟进负载来调整，当系统负荷较高时， Innodb 动态调整 sleep 时间可使得数据库稳定运行。

## innodb_commit_concurrency

该值只能为默认值 0 ， mysql 不限制并发提交。大于 0 表示允许 N 个事务在同一时间点提交， N 的范围是 0-1000 。注意事项： mysqld 运行时，不许把 innodb_commit_concurrency 的值从 0 改为非 0 ，或非 0 的值改为 0 ；但允许从 N 改为 M （ N 及 M 均大于 0 ）。

## innodb_concurrency_tickets

当请求被 innodb 接受的时候，会获得一个消费凭证 innodb_concurrency_tickets（ mysql 版本 v5.5 默认 500 ， v5.6 和 v5.7 默认 5000 ），当这个请求中有多个SQL被执行的时候，每执行一次，消费一次 tickets ，在次数用完之前，该线程重新请求时无须再进行前面 thread 是否达到并发限制值的检查。如果 innodb_concurrency_tickets设 置小些，适用于小事物操作较多的系统，可以快速使用完线程后退出来，提供给其他请求使用；而对于大事务来说，可能会循环进入等待队列中等待执行完成，这会耗费更多时间及资源；如果 innodb_concurrency_tickets 设置大些，适用于大事务频繁操作的系统，这样大事务则不需要频繁进入 queue 等待队列，可以通过较少的请求来处理；但是对于小事务来说，则意味着他们要等待更长的时候，才能排队进入到内核执行。所以，当 innodb_thread_concurrency>0 时，需要上下调整 innodb_concurrency_tickets ，使其达到最佳性能。可以通过 **show engine innodb status的queue** 查看，也可以通过 **information_schema.INNODB_TRX 表中的 trx_concurrency_tickets 字段值**查看消费次数情况。

## transaction_isolation

## table_open_cache

可设置成 2048 。表的缓存： 2*max_connections-5*max_connections ，但是不能大于操作系统文件描述符限制。当某一连接访问一个表时， MySQL 会检查当前已缓存表的数量。如果该表已经在缓存中打开，则会直接访问缓存中的表已加快查询速度；如果该表未被缓存，则会将当前的表添加进缓存并进行查询。

## query_cache_size、query_cache_type

默认 0 ，建议设置成？。

Query Cache（查询缓存，以下简称 QC ）存储 SELECT 语句及其产生的数据结果，特别适用于：频繁提交同一个语句，并且该表数据变化不是很频繁的场景，例如一些静态页面，或者页面中的某块不经常发生变化的信息。 QC 有可能会从 InnoDB Buffer Pool 或者 MyISAM key buffer 里读取结果。

由于 QC 需要缓存最新数据结果，因此表数据发生任何变化（ INSERT、UPDATE、DELETE 或其他可能产生数据变化的操作），都会导致 QC 被刷新。 QC 严格要求 2 次 SQL 请求要完全一样，包括 SQL 语句，连接的数据库、协议版本、字符集等因素都会影响。

如果线上环境中 99% 以上都是只读，很少有更新，才考虑开启 QC ，否则建议关闭（设置选项 query_cache_type=0 和 query_cache_size=0 ）。

- 查询缓存能够加速已经存在缓存的查询语句的速度，可以不用重新解析和执行而获得正确的记录集；
- 查询缓存中涉及的表，每一个表对象都有一个属于自己的全局性质的锁；
- 表若是做 DDL、FLUSH TABLES 等类似操作，触发相关表的查询缓存信息清空；
- 表对象的 DML 操作，必须优先判断是否需要清理相关查询缓存的记录信息，将不可避免地出现锁等待事件；
- 查询缓存的内存分配问题，不可避免地产生一些内存碎片；
- 查询缓存对是否是一样的查询语句，要求非常苛刻，而且还不智能；

---

- [MySQL 教程合集](https://mp.weixin.qq.com/s/jflrWU62pBtevS62lEIHkQ)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)