---
title: MySQL 安装 - Windows
date: 2018-07-28 09:16:23
categories: MySQL
tags: [MySQL, 安装]
toc: true
# thumbnail: /images/mysql/mysql-thumbnail.jpg
---
手把手带你在 Windows 上安装 MySQL-5.6 。
<!-- more -->

```bash
# 下载免安装版本的 MySQL
# https://downloads.mysql.com/archives/community/
# mysql-5.6.42-winx64.zip

# 解压并准备配置文件 my.ini 放到下列目录中
D:\mysql-5.6.42-3306

# 修改配置文件 my.ini
[client]
port = 3306
#socket = D:\mysqldata\tmp\mysql.sock
default-character-set = utf8

[mysql]
default-character-set = utf8
prompt = "\\u:\\d> "
auto-rehash

[mysqld]
port = 3306
basedir = D:\mysql-5.6.42-3306
datadir = D:\mysql-5.6.42-3306\data
#socket = D:\mysql-5.6.42-3306\tmp\mysql.sock
#tmpdir = D:\mysql-5.6.42-3306\tmp
#log_bin = D:\mysql-5.6.42-3306\log\mysql-bin.log
#relay_log = D:\mysql-5.6.42-3306\log\mysql-relay-bin.log
#log_error = D:\mysql-5.6.42-3306\log\alert.log
#slow_query_log_file = D:\mysql-5.6.42-3306\log\mysql_slow.log
default-time-zone = '+8:00'
character-set-server = utf8
collation-server = utf8_unicode_ci
init_connect = 'SET NAMES utf8'
max_connections = 1000
max_user_connections = 1000
max_connect_errors = 90000000
max_allowed_packet = 16M
back_log = 5000
wait_timeout = 120
interactive_timeout = 120
sort_buffer_size = 2M
join_buffer_size = 2M
server-id = 1
default-storage-engine = innodb
innodb-file-per-table = 1
......

# 初始化
d:
cd mysql-5.6.42-3306/bin
mysqld.exe --initialize

# 将 mysql 从 windows 服务中移除
mysqld.exe --remove MySQL-3306

# 将 mysql 添加到 windows 服务中
mysqld.exe --install MySQL-3306 --defaults-file="D:\mysql-5.6.42-3306\my.ini"

# 启动服务
net start MySQL-3306

# 登录并修改密码（有些时候需要初始密码）
cd d:/mysql-5.6.42-3306/bin
mysql -uroot
# mysql -uroot -p
update mysql.user set password = password('123456') where user='root';
# alter user 'root'@'localhost' identified by '123456';
flush privileges;

# 授权远程访问
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
flush privileges;

# 关闭服务
net stop MySQL-3306

# 根据端口查询 pid
netstat -ano|findstr "3306"

# 根据 pid 查询进程名
tasklist|findstr "9452"

# 杀进程
taskkill /f /pid 9452
```

---

- [MySQL 教程合集](https://mp.weixin.qq.com/s/jflrWU62pBtevS62lEIHkQ)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)