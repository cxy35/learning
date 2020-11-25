---
title: MySQL 用户与权限
date: 2018-09-01 17:48:17
categories: MySQL
tags: [MySQL]
toc: true
---
学习 MySQL 的用户管理与权限管理。
<!-- more -->

## MySQL 用户

```sql
-- 创建用户 'test'@'%'
CREATE USER test IDENTIFIED BY '123456';
CREATE USER 'test'@'%' IDENTIFIED BY '123456';
-- 创建用户 'test'@'localhost'
CREATE USER 'test'@'localhost' IDENTIFIED BY '123456';
flush privileges;

-- 删除用户 'test'@'%'
drop user 'test'@'%';
-- 删除用户 'test'@'localhost'
drop user 'test'@'localhost';
flush privileges;
```

## MySQL 权限

```sql
-- 权限汇总
-- SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, SHUTDOWN, PROCESS, FILE, REFERENCES, INDEX, ALTER, SHOW DATABASES, SUPER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER 

-- 授权/取消，单个权限
GRANT SELECT ON mydbname.mytbname TO 'test'@'%';
REVOKE SELECT ON mydbname.mytbname FROM 'test'@'%';
GRANT SELECT ON mydbname.* TO 'test'@'%';
REVOKE SELECT ON mydbname.* FROM 'test'@'%';
flush privileges;

-- 授权/取消，多个权限
GRANT SELECT, INSERT ON mydbname.mytbname TO 'test'@'%';
REVOKE SELECT, INSERT ON mydbname.mytbname FROM 'test'@'%';
flush privileges;

-- 授权/取消，所有权限
GRANT ALL PRIVILEGES ON mydbname.mytbname TO 'test'@'%';
REVOKE ALL PRIVILEGES ON mydbname.mytbname FROM 'test'@'%';
flush privileges;

-- 查看已授权的权限
show grants for 'test'@'%'; 
show grants for 'test'@'localhost'; 
```

|权限|说明|
|:-|:-|
|all| |
|alter| |
|alter routine|使用alter procedure和drop procedure|
|create| |
|create routine|使用create  procedure|
|create temporary tables|使用create temporary table|
|create user| |
|create view| |
|delete| |
|drop| |
|execute|使用call和存储过程|
|file|使用select into outfile和load data infile|
|grant option|使用grant和revoke|
|index|使用create index和drop index|
|insert| |
|lock tables|锁表|
|process|使用show full processlist|
|reload|使用flush|
|replication client|服务器位置访问|
|replication slave|由复制从属使用|
|select| |
|show databases| |
|show view| |
|shutdown|使用mysqladmin shutdown来关闭mysql|
|super| |
|update| |
|usage|无访问权限|

---

- [MySQL 教程合集](https://mp.weixin.qq.com/s/jflrWU62pBtevS62lEIHkQ)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)