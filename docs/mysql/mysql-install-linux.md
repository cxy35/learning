手把手带你在 Linux 上安装 MySQL-5.6/5.7 。
<!-- more -->

## 1 准备工作

### 1.1 确认系统环境

```bash
# 查看 Linux 版本
cat /etc/issue
# CentOS release 6.5 (Final)
# Kernel \r on an \m

# 查看内核版本
cat /proc/version
# Linux version 2.6.32-431.el6.x86_64 (mockbuild@c6b8.bsys.dev.centos.org) (gcc version 4.4.7 20120313 (Red Hat 4.4.7-4) (GCC) ) #1 SMP Fri Nov 22 03:15:09 UTC 2013

# 查看 CentOS 版本
cat /etc/centos-release
# CentOS Linux release 7.9.2009 (Core)
```

### 1.2 删除 MySQL 历史版本

```bash
# 检查是否有历史版本
rpm -qa|grep -i mysql  ##-i选项表示匹配时忽略大小写
# mysql-5.1.73-8.el6_8.x86_64
# mysql-devel-5.1.73-8.el6_8.x86_64
# mysql-libs-5.1.73-8.el6_8.x86_64

# 删除，--nodeps选项表示忽略依赖关系
rpm -e mysql-5.1.73-8.el6_8.x86_64 --nodeps
rpm -e mysql-devel-5.1.73-8.el6_8.x86_64 --nodeps
rpm -e mysql-libs-5.1.73-8.el6_8.x86_64 --nodeps


# 查找mysql相关目录
find / -name mysql
# /usr/local/mysql
# /usr/share/mysql
# /usr/lib64/mysql
# /var/lib/mysql

# 删除
rm -rf /usr/local/mysql /usr/share/mysql /usr/lib64/mysql /var/lib/mysql
```

### 1.3 下载安装包

MySQL 下载地址：[https://downloads.mysql.com/archives/community/](https://downloads.mysql.com/archives/community/)

- 通用安装（**推荐**）：`Operating System:` 下拉项选择 `Linux - Generic` ，然后下载后缀为 `.tar.gz` 的二进制文件，如 `mysql-5.6.42-linux-glibc2.12-x86_64.tar.gz` 或 `mysql-5.7.29-linux-glibc2.12-x86_64.tar.gz` 。上传到服务器的 `/usr/local/mydata/temp` 目录下。如果没有，则手动新建。
- rpm 安装：如 `MySQL-5.6.42-1.el6.x86_64.rpm-bundle.tar` 。
- yum 安装。
- 源码编译安装：`Operating System:` 下拉项选择 `Source Code` ，然后下载后缀为 `.tar.gz` 的源码文件，如 `mysql-5.6.42.tar.gz` 或 `mysql-boost-5.7.29.tar.gz` 。上传到服务器的 `/usr/local/mydata/temp` 目录下。

## 2 安装 MySQL

### 2.1 通用安装（**推荐**）

- 上传 `mysql-5.6.42-linux-glibc2.12-x86_64.tar.gz` 或 `mysql-5.7.29-linux-glibc2.12-x86_64.tar.gz` 。


- 将二进制文件解压到 `/usr/local/mydata/soft` ，并重命名为 mysql 。

```bash
# 其中 mysql-5.6.42-linux2.6-x86_64.tar.gz 换成实际的名称
cd /usr/local/mydata/temp
tar -xzvf mysql-5.6.42-linux2.6-x86_64.tar.gz -C /usr/local/mydata/soft
cd /usr/local/mydata/soft
mv mysql-5.6.42-linux2.6-x86_64 mysql
```

- 创建数据库实例的相关目录和文件

```bash
mkdir -p /usr/local/mydata/soft/mysql/{data,etc,log,tmp}
touch /usr/local/mydata/soft/mysql/log/alert.log
touch /usr/local/mydata/soft/mysql/log/mysql_slow.log
```

- 添加 mysql 组和 mysql 用户，用于设置 mysql 安装目录文件所有者和所属组。

```bash
groupadd mysql

# -r 参数表示 mysql 用户是系统用户，不可用于登录系统
useradd -r -g mysql mysql
```

- 复制并修改配置文件。

```bash
# 下面复制的是最简单最低要求的一个配置文件（可根据实际需要拷贝不同的配置文件），具体的参数配置优化见下文
# cp /usr/local/mydata/soft/mysql/support-files/my-small.cnf /usr/local/mydata/soft/mysql/etc/my.cnf

vi /usr/local/mydata/soft/mysql/etc/my.cnf

# MySQL-5.6主从，主库1（CPU 32核，内存 64G）
[client]
port = 3306
socket = /usr/local/mydata/soft/mysql/tmp/mysql.sock
default-character-set = utf8

[mysql]
default-character-set = utf8
prompt = "\\u:\\d> "
auto-rehash

[mysqld]
port = 3306
basedir = /usr/local/mydata/soft/mysql
socket = /usr/local/mydata/soft/mysql/tmp/mysql.sock
tmpdir = /usr/local/mydata/soft/mysql/tmp
datadir = /usr/local/mydata/soft/mysql/data
log-bin = /usr/local/mydata/soft/mysql/log/mysql-bin.log
relay_log = /usr/local/mydata/soft/mysql/log/mysql-relay-bin.log
log-error = /usr/local/mydata/soft/mysql/log/alert.log
slow-query-log-file = /usr/local/mydata/soft/mysql/log/mysql_slow.log
default-time-zone = '+8:00'
character-set-server = utf8
collation-server = utf8_unicode_ci
init_connect = 'SET NAMES utf8'
read_only = 0
relay_log_purge = 0
skip-name-resolve
skip-external-locking
max_connections = 4000
max_user_connections = 4000
max_connect_errors = 90000000
max_allowed_packet = 16M
back_log = 5000
wait_timeout = 120
interactive_timeout = 120
sort_buffer_size = 2M
join_buffer_size = 2M
thread_cache_size = 32
tmp_table_size = 256M
max_heap_table_size = 256M
query_cache_type = 0
key_buffer_size = 128M
read_buffer_size = 2M
read_rnd_buffer_size = 8M
lower_case_table_names = 1
bulk_insert_buffer_size = 16M
#sql_mode = NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
sql_mode = ""
explicit_defaults_for_timestamp = true
server-id = 1
sync_binlog = 100
log_slave_updates = 1
relay_log_info_repository = TABLE
master_info_repository = TABLE
relay_log_recovery = 1
relay_log_purge = ON
binlog_format =MIXED 
binlog_cache_size = 1G
max_binlog_cache_size = 1G
max_binlog_size = 1G
expire_logs_days = 7
long_query_time = 10
slow-query-log = 1
default-storage-engine = innodb
innodb-fast-shutdown = 1
innodb-force-recovery = 0
innodb_buffer_pool_size = 40G
innodb_buffer_pool_instances = 2
innodb_buffer_pool_dump_at_shutdown = 1
innodb-file-per-table = 1
innodb-write-io-threads = 16
innodb-read-io-threads = 16
innodb-thread-concurrency = 32
innodb-flush-log-at-trx-commit = 2
innodb_log_buffer_size = 8M
innodb_log_files_in_group = 4
innodb-max-dirty-pages-pct = 80
innodb-lock-wait-timeout = 120
innodb_flush_method = O_DIRECT
innodb_log_file_size = 10M
innodb_data_file_path = ibdata1:10M:autoextend
transaction_isolation = READ-COMMITTED
innodb_io_capacity_max = 10000
innodb_io_capacity = 6000
innodb_lru_scan_depth = 8000
innodb_file_format = Barracuda
innodb_file_format_max = Barracuda
#replicate-ignore-db = mysql
#replicate-ignore-db = information_schema
#replicate-ignore-db = test
#replicate-ignore-db = performance_schema
federated

[mysqldump]
quick
max_allowed_packet = 16M

[isamchk]
key_buffer = 128M
sort_buffer_size = 2M
read_buffer = 8M
write_buffer = 8M

[myisamchk]
key_buffer = 128M
sort_buffer_size = 2M
read_buffer = 8M
write_buffer = 8M

[mysqlhotcopy]
interactive-timeout

[mysqld_safe]
open-files-limit = 8192
```

- 初始化 mysql 数据库实例。

```bash
# 进入 mysql 目录
cd /usr/local/mydata/soft/mysql

# 修改 mysql 目录与文件的所有者为 mysql，必须要 mysql 用户的才能进行后续安装（目录权限，重要！！！）
chown -R mysql:mysql /usr/local/mydata/soft/mysql

# 初始化数据库实例，会对 mysql 中的 data 目录进行初始化并创建一些系统表格
# --user 指定运行 mysqld 进程的用户，如：mysql。
# --basedir 指定 mysql 的安装目录，如：/usr/local/mydata/soft/mysql，也可在配置文件中指定。
# --datadir 指定 mysql 的数据存放目录，如：/usr/local/mydata/soft/mysql/data，也可在配置文件中指定。
# --defaults-file 指定配置文件，如：/usr/local/mydata/soft/mysql/etc/my.cnf。
###### V5.6 初始化命令 ######
/usr/local/mydata/soft/mysql/scripts/mysql_install_db --user=mysql --basedir=/usr/local/mydata/soft/mysql --datadir=/usr/local/mydata/soft/mysql/data --defaults-file=/usr/local/mydata/soft/mysql/etc/my.cnf
###### V5.6 初始化命令 ######

###### V5.7 初始化命令 ######
/usr/local/mydata/soft/mysql/bin/mysqld --user=mysql --basedir=/usr/local/mydata/soft/mysql --datadir=/usr/local/mydata/soft/mysql/data --initialize
# 会生成一个临时密码，拷贝出来
# A temporary password is generated for root@localhost: Uk*ui4)!,sM+
###### V5.7 初始化命令 ######

# 修改 mysql 目录与文件的所有者为 mysql（目录权限，重要！！！）
chown -R mysql:mysql /usr/local/mydata/soft/mysql
```

- 启动 mysql（如果需要可通过将 mysql 配置成服务注册开机启动）。

```bash
# 服务名为 mysql，这样就可以使用 service mysql 命令启动/停止服务，非必须
# cp /usr/local/mydata/soft/mysql/support-files/mysql.server /etc/init.d/mysql

# 把 mysql 服务注册为开机启动，非必须
# chkconfig --add mysql

# 查看是否添加成功，非必须
# chkconfig --list mysql

# 启动 start /停止 stop /重启 restart /查看状态 status，不推荐
# service mysql start

# 也可以用下面的方法启动，不推荐
# /usr/local/mydata/soft/mysql/support-files/mysql.server start

# 推荐用下面的方法启动，推荐！！！
# --read_only=1  # 打开只读，从库设置
# --skip-slave-start  # 不启动主从
# set global read_only=0;  # 关闭只读，主库需要
/bin/bash /usr/local/mydata/soft/mysql/bin/mysqld_safe --defaults-file=/usr/local/mydata/soft/mysql/etc/my.cnf &
# /bin/bash /usr/local/mydata/soft/mysql/bin/mysqld_safe --defaults-file=/usr/local/mydata/soft/mysql/etc/my.cnf --read_only=1 &

# 推荐用下面的方法停止，推荐！！！
/usr/local/mydata/soft/mysql/bin/mysqladmin -uroot -p -S /usr/local/mydata/soft/mysql/tmp/mysql.sock shutdown
# 123456

# 推荐用下面的方法登录，推荐！！！
###### V5.6 默认无密码 ######
###### V5.7 初始化的时候会生成一个临时密码，先拷贝出来 ######
/usr/local/mydata/soft/mysql/bin/mysql -uroot -p -S /usr/local/mydata/soft/mysql/tmp/mysql.sock
# 123456

# 检查是否启动
ps -ef|grep mysql 或 netstat -anp|grep mysql
root      2248     1  0 Jun15 ?        00:00:00 /bin/bash /usr/local/mydata/soft/mysql/bin/mysqld_safe --defaults-file=/usr/local/mydata/soft/mysql/etc/my.cnf --read_only=1
mysql     3344  2248 19 Jun15 ?        3-22:41:14 /usr/local/mydata/soft/mysql/bin/mysqld --defaults-file=/usr/local/mydata/soft/mysql/etc/my.cnf --basedir=/usr/local/mydata/soft/mysql --datadir=/usr/local/mydata/soft/mysql/data --plugin-dir=/usr/local/mydata/soft/mysql/lib/plugin --user=mysql --read-only=1 --log-error=/usr/local/mydata/soft/mysql/log/alert.log --open-files-limit=8192 --pid-file=localhost.localdomain.pid --socket=/usr/local/mydata/soft/mysql/tmp/mysql.sock --port=3306
root     30497  1836  0 16:57 pts/0    00:00:00 grep mysql
```

### 2.2 rpm 安装

- 上传 `MySQL-5.6.42-1.el6.x86_64.rpm-bundle.tar` 。

- 将 rpm 安装包解压到 /usr/local/mydata/soft/mysql。

```bash
tar -xvf MySQL-5.6.42-1.el6.x86_64.rpm-bundle.tar -C /usr/local/mydata/soft/mysql

# MySQL-client-5.6.42-1.el6.x86_64.rpm
# MySQL-devel-5.6.42-1.el6.x86_64.rpm
# MySQL-embedded-5.6.42-1.el6.x86_64.rpm
# MySQL-server-5.6.42-1.el6.x86_64.rpm
# MySQL-shared-5.6.42-1.el6.x86_64.rpm
# MySQL-shared-compat-5.6.42-1.el6.x86_64.rpm
# MySQL-test-5.6.42-1.el6.x86_64.rpm
```
- 安装。

```bash
# 可能需要给文件增加执行权限
# chmod a+x *.rpm
rpm -ivh *.rpm
# rpm -ivh MySQL-5.6.42-1.el6.x86_64.rpm
```

- 如果报错，则对应解决。

```bash
# 报错：
file /usr/share/mysql/charsets/cp1251.xml from install of MySQL-5.6.42-1.el6.x86_64 conflicts with file from package mysql-libs-5.1.52-1.el6_0.1.i686
# 安装包冲突，卸载安装包
# -y 的意思就是不用询问是否 remove
yum -y remove remove mysql-libs-5.1.52*

# 报错：
error: Failed dependencies:
libc.so.6 is needed by MySQL-server-community-5.1.63-1.rhel4.i386
libc.so.6(GLIBC_2.0) is needed by MySQL-server-community-5.1.63-1.rhel4.i386
libc.so.6(GLIBC_2.1) is needed by MySQL-server-community-5.1.63-1.rhel4.i386
# 缺少相关包，安装相关包
yum install libc.so.6
```

- 再次安装。

```bash
rpm -ivh *.rpm
# rpm -ivh MySQL-5.6.42-1.el6.x86_64.rpm
```

- 启动服务。

```bash
# 不是原生的systemctl服务，建议使用 service
service start mysql
chkconfig --list mysql
chkconfig mysql on

# systemctl start mysql
# systemctl stop mysql

# 检查是否启动
ps -ef|grep mysql 或 netstat -anp|grep mysql
tcp        0      0 0.0.0.0:3306                0.0.0.0:*                   LISTEN      6602/mysqld         
unix  2      [ ACC ]     STREAM     LISTENING     23737  6602/mysqld         /usr/local/mydata/soft/mysql/data/mysql.sock
```

### 2.3 yum 安装

- 安装。

```bash
yum install -y mysql-server mysql mysql-devel
```

- 如果报错，则对应解决。

```bash
# 报错：
file /usr/share/mysql/ukrainian/errmsg.sys from install of MySQL-server-5.5.18-1.rhel5.i386 conflicts with file from package mysql-libs-5.0.46-1.rhel5.i386
# 卸载安装包
# -y 的意思就是不用询问是否 remove
yum -y remove mysql-libs-5.0.46-1.rhel5.i386
```

- 再次安装。

```bash
yum install -y mysql-server mysql mysql-devel
```

- 启动服务。

```bash
service mysql start

# 检查是否启动
ps -ef|grep mysql 或 netstat -anp|grep mysql
tcp        0      0 0.0.0.0:3306                0.0.0.0:*                   LISTEN      6602/mysqld         
unix  2      [ ACC ]     STREAM     LISTENING     23737  6602/mysqld         /usr/local/mydata/soft/mysql/data/mysql.sock
```

### 2.4 源码编译安装

> 这种安装方式一般适用于特殊的操作系统（如：银河麒麟 V10 等），采用的 Linux 内核是 `aarch64`，MySQL 官方可能没有提供编译好的二进制文件，所以需要自己编译安装。

- 上传 `mysql-5.6.42.tar.gz` 或 `mysql-boost-5.7.29.tar.gz` 。

- 安装相关依赖环境

```bash
yum install -y ncurses-devel rpcgen cmake gcc gcc-c++ bison bison-devel  \
ncurses autoconf automake libaio-devel bison perl-Time-HiRes libatomic \
openssl-devel.aarch64 openssl-devel.aarch64 libtirpc libtirpc-devel
```

- 解压源码文件。

```bash
# 其中 mysql-boost-5.7.29.tar.gz 换成实际的名称
cd /usr/local/mydata/temp
tar -xzvf mysql-boost-5.7.29.tar.gz
```

- 编译

```bash
cd /usr/local/mydata/temp/mysql-5.7.29

cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mydata/soft/mysql  \
-DMYSQL_DATADIR=/usr/local/mydata/soft/mysql/data \
-DSYSCONFDIR=/usr/local/mydata/soft/mysql/etc \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DWITH_FEDERATED_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DENABLED_LOCAL_INFILE=1 \
-DENABLE_DTRACE=0 \
-DDEFAULT_CHARSET=utf8mb4 \
-DDEFAULT_COLLATION=utf8mb4_general_ci \
-DWITH_EMBEDDED_SERVER=1 \
-DDOWNLOAD_BOOST=1 \
-DWITH_BOOST=/usr/local/mydata/temp/mysql-5.7.29/boost

# 创建上述相关目录
mkdir -p /usr/local/mydata/soft/mysql/{data,etc,log,tmp}

# 编译
make && make install

# 执行完成之后，在 /usr/local/mydata/soft/mysql 目录下多了一些 MySQL 相关目录。
```

- 下面开始安装，具体安装方法见**通用安装**，从*创建数据库实例的相关目录和文件*章节开始往后。

## 3 配置 MySQL

### 3.1 设置 root 密码

```bash
# 先登录，再执行
mysql> alter user 'root'@'localhost' identified by '123456';

# 或
/usr/local/mydata/soft/mysql/bin/mysqladmin -u root password '123456' -S /usr/local/mydata/soft/mysql/tmp/mysql.sock
```

### 3.2 登录

```bash
# 登录 mysql
# /usr/local/mydata/soft/mysql/bin/mysql -uroot -p
/usr/local/mydata/soft/mysql/bin/mysql -uroot -p -S /usr/local/mydata/soft/mysql/tmp/mysql.sock

# 或者增加 my 快捷登录方式，非必须
# vim /usr/local/bin/my
# 加入以下内容：/usr/local/mydata/soft/mysql/bin/mysql -uroot -p123456 -S /usr/local/mydata/soft/mysql/tmp/mysql.sock
# chmod +x /usr/local/bin/my
# /usr/local/bin/my
```

### 3.3 授权远程访问

```bash
# 配置授权
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
mysql> flush privileges;
```

### 3.4 新建用户并授权

```bash
# 新建 grid 用户并授权（无 DROP 权限），用于业务操作。因为 root 用户对从库的只读设置无效，操作有风险。
# 开放某个库的所有表：`mydbname`.*，开放所有库的所有表：*.*
mysql> CREATE USER 'grid'@'%' IDENTIFIED BY 'grid@123456';
mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, INDEX, ALTER ON `mydbname`.* TO 'grid'@'%';
mysql> flush privileges;
```

### 3.5 参数配置优化

- [MySQL 参数配置优化](docs/mysql/mysql-cnf.md)
- [MySQL 配置文件 - 5.6](https://github.com/cxy35/collect/tree/master/mysql/my.cnf)

```bash
vi /usr/local/mydata/soft/mysql/etc/my.cnf

[client]
port = 3306
socket = /usr/local/mydata/soft/mysql/data/mysql.sock

[mysqld]
port = 3306
socket = /usr/local/mydata/soft/mysql/data/mysql.sock

basedir = /usr/local/mydata/soft/mysql
datadir = /usr/local/mydata/soft/mysql/data
tmpdir = /usr/local/mydata/soft/mysql/tmp

# 字符集
character_set_server = utf8
init_connect = 'SET NAMES utf8'
# Windows中默认1，Linux中默认0。表名忽略大小写。1=忽略，0=不忽略。
lower_case_table_names = 1

# 最大连接数
max_connections = 2000
# 防止数据导入时内容太大导致无法导入的问题
max_allowed_packet = 16M
innodb_buffer_pool_size = 40G
join_buffer_size = 2M
sort_buffer_size = 2M
read_rnd_buffer_size = 8M
wait_timeout = 120
interactive_timeout = 120
sync_binlog = 100

# ......
```

---

- [MySQL 教程合集](https://mp.weixin.qq.com/s/jflrWU62pBtevS62lEIHkQ)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.top](https://cxy35.top)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)