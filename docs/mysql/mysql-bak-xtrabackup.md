---
title: MySQL 备份与恢复 - xtrabackup
date: 2018-10-14 15:46:37
categories: MySQL
tags: [MySQL, 备份, xtrabackup]
toc: true
---
【TODO-未整理、未实测】MySQL 备份与恢复 - xtrabackup 。
<!-- more -->

数据库备份恢复之XtraBackup概述及安装部署  
一、xtrabackup工具介绍及备份过程概述  
1.xtrabackup简介：  
mysqldump备份方式是采用逻辑备份，其最大的缺陷就是备份和恢复速度都慢，对于一个小于50G的数据库而言，这个速度还是能够接受的，如果数据库非常大，那再使用mysqldump备份就不太适合了。  
Xtrabackup是由percona提供的mysql数据库备份工具，据官方介绍，这也是世界上唯一一个开源的能够对innodb和xtradb数据库进行物理热备的工具。  
2.xtrabakcup特点：  
1）备份过程快速，可靠；  
　　2）备份过程不会打断正在执行的事务（不需要锁表）  
　　3）能够给予压缩等功能节约磁盘空间和流量。  
　　4）自动实现备份检验；  
　　5）还原速度快；  
　　6）可以进行流传出备份，备份到另外一台机器上。  
Xtrabackup中主要有包含两个工具：  
1.innobackupex：是将xtrabackup进行封装的perl脚本，提供了备份myisam表的能力，  
2.xtrabackup：是用于热备innodb，xtradb表中数据的工具，不能备份其他类型的表，也不能备份数据表结构；

【xtrabackup 全量备份恢复】  
1. 完全备份  
创建用于备份恢复的用户 pxb 并赋予权限  
create user pxb@'localhost' identified by '123456';  
grant reload,process,lock tables,replication client on *.* to pxb[@localhost](https://my.oschina.net/u/570656);

创建存放目录  
mkdir -pv /data/pxb

进行数据库全备（生成全备：/data/pxb/2017-04-24_02-46-11）  
innobackupex --defaults-file=/etc/my.cnf --user=pxb --password=123456 --socket=/tmp/mysql.sock  /data/pxb

2. 全备恢复  
关闭数据库并删除数据文件  
/etc/init.d/mysqld stop  
cd /home/mysql  
mv data data_bak  
mkdir data

准备(prepare)一个完全备份：--apply-log ( /data/pxb/2017-04-24\_02-46-11/ 为备份目录，执行之后 xtrabackup\_checkpoints 文件中的 backup_type = full-prepared )  
innobackupex --apply-log /data/pxb/2017-04-24_02-46-11/

执行恢复操作：  
innobackupex  --defaults-file=/etc/my.cnf --copy-back --rsync /data/pxb/2017-04-24_02-46-11/

更改 data/ 目录权限并启动mysql：  
chown -R mysql:mysql data/  
/etc/init.d/mysqld start

验证

  
【xtrabackup 增量备份恢复】  
我们以之前做的全备为基准，在其基础上做增量备份：  
增量备份1：（以全备为基准：/data/pxb/2017-04-24\_02-46-11/）（生成增量1：/data/pxb/inc/2017-04-28\_01-09-40）  
innobackupex --defaults-file=/etc/my.cnf --user=pxb --password=123456 --socket=/tmp/mysql.sock --incremental /data/pxb/inc --incremental-basedir=/data/pxb/2017-04-24_02-46-11/ --parallel=2

增量备份2：（以增量1为基准：/data/pxb/inc/2017-04-28\_01-09-40/）（生成增量2：/data/pxb/inc/2017-04-28\_01-27-46）  
innobackupex --defaults-file=/etc/my.cnf --user=pxb --password=123456 --socket=/tmp/mysql.sock --incremental /data/pxb/inc --incremental-basedir=/data/pxb/inc/2017-04-28_01-09-40/ --parallel=2

增量备份的恢复  
增量备份的恢复需要有3个步骤  
1恢复完全备份  
2恢复增量备份到完全备份(开始恢复的增量备份要添加--redo-only参数，到最后一次增量备份要去掉--redo-only)  
3对整体的完全备份进行恢复，回滚未提交的数据  
##准备一个全备##  
innobackupex --apply-log --redo-only /data/pxb/2017-04-24_02-46-11/

##将增量1应用到完全备份##  
innobackupex --apply-log --redo-only /data/pxb/2017-04-24\_02-46-11/ --incremental-dir=/data/pxb/inc/2017-04-28\_01-09-40/

##将增量2应用到完全备份，注意不加 --redo-only 参数了##  
innobackupex --apply-log /data/pxb/2017-04-24\_02-46-11/ --incremental-dir=/data/pxb/inc/2017-04-28\_01-27-46/

##把所有合在一起的完全备份整体进行一次apply操作，回滚未提交的数据##  
innobackupex --apply-log /data/pxb/2017-04-24_02-46-11/

关闭数据库并删除数据文件  
/etc/init.d/mysqld stop  
cd /home/mysql  
mv data data_bak2  
mkdir data

执行恢复操作：  
innobackupex --defaults-file=/etc/my.cnf --copy-back --rsync /data/pxb/2017-04-24_02-46-11/

更改 data/ 目录权限并启动mysql：  
chown -R mysql:mysql data/  
/etc/init.d/mysqld start

验证

---

- [MySQL 教程合集](https://mp.weixin.qq.com/s/jflrWU62pBtevS62lEIHkQ)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)