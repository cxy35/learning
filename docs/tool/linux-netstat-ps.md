---
title: Linux 下查看某个端口被哪个进程或程序占用
date: 2018-12-16 16:28:26
categories: Linux
tags: [Linux]
toc: true
---
Linux 下查看某个端口被哪个进程或程序占用。
<!-- more -->

```bash
[root@dbserver ~]$ netstat -anp|grep 3306
tcp        0      0 0.0.0.0:3306                0.0.0.0:*                   LISTEN      3005/mysqld         
tcp        0      0 192.168.71.62:3306          192.168.71.98:57629         ESTABLISHED 3005/mysqld         
tcp        0      0 192.168.71.62:3306          192.168.11.118:57895        ESTABLISHED 3005/mysqld         
tcp        0      0 192.168.71.62:3306          192.168.71.98:57639         ESTABLISHED 3005/mysqld         
tcp        0      0 192.168.71.62:3306          192.168.71.97:57139         ESTABLISHED 3005/mysqld                                   
tcp        0      0 192.168.71.62:3306          192.168.71.56:57896         ESTABLISHED 3005/mysqld                           
tcp        0      0 192.168.71.62:3306          192.168.71.35:58646         ESTABLISHED 3005/mysqld         
tcp        0      0 192.168.71.62:3306          192.168.71.35:58629         ESTABLISHED 3005/mysqld         
tcp        0      0 192.168.71.62:3306          192.168.71.97:57132         ESTABLISHED 3005/mysqld         
unix  2      [ ACC ]     STREAM     LISTENING     13125  3005/mysqld         /usr/local/mysql3306/data/mysqld.sock
```

由上面可以看到端口 3306 对应的进程 pid=3005 ，进程为 mysqld 。

再执行 **kill -9 3005** 可结束该进程。

```bash
[root@dbserver ~]$ ps -ef|grep mysqld
root      2123     1  0 09:39 ?        00:00:00 /bin/sh /usr/local/mysql3306/bin/mysqld_safe --datadir=/usr/local/mysql3306/data --pid-file=/usr/local/mysql3306/data/dbserver.pid
mysql     3005  2123  1 09:39 ?        00:03:40 /usr/local/mysql3306/bin/mysqld --basedir=/usr/local/mysql3306 --datadir=/usr/local/mysql3306/data --plugin-dir=/usr/local/mysql3306/lib/plugin --user=mysql --log-error=/usr/local/mysql3306/data/error.log --open-files-limit=8192 --pid-file=/usr/local/mysql3306/data/dbserver.pid --socket=/usr/local/mysql3306/data/mysqld.sock --port=3306
root      4186  3632  0 14:23 pts/0    00:00:00 grep mysqld
```

由上面可以看到进程 pid=3005 对应的 mysqld 进程的具体信息。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)