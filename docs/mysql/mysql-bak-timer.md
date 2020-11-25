---
title: MySQL 定时备份
date: 2018-10-20 09:35:54
categories: MySQL
tags: [MySQL, 备份]
toc: true
---
在 Linux 上实现 MySQL 定时备份。
<!-- more -->

## 1 准备备份的脚本文件

### 1.1 创建脚本文件

```bash
vi /usr/local/mysqldata/bak/bak.sh

# !/bin/bash
# db-export variables
EXPORT_DB_HOST="127.0.0.1"
EXPORT_DB_PORT="3306"
EXPORT_DB_USER="root"
EXPORT_DB_PASSWORD="123456"
EXPORT_DB_NAME="db1"
EXPORT_DB_CMD="/usr/local/mysql/bin/mysqldump"
EXPORT_CMD_PREFIX="$EXPORT_DB_CMD -h$EXPORT_DB_HOST -P$EXPORT_DB_PORT -u$EXPORT_DB_USER -p$EXPORT_DB_PASSWORD $EXPORT_DB_NAME"

# db-import variables
IMPORT_DB_HOST="127.0.0.1"
IMPORT_DB_PORT="3306"
IMPORT_DB_USER="root"
IMPORT_DB_PASSWORD="123456"
IMPORT_DB_NAME="db2"
IMPORT_DB_CMD="/usr/local/mysql/bin/mysql"
IMPORT_CMD_PREFIX="$IMPORT_DB_CMD -h$IMPORT_DB_HOST -P$IMPORT_DB_PORT -u$IMPORT_DB_USER -p$IMPORT_DB_PASSWORD $IMPORT_DB_NAME"

# bak variables
# BAK_FILE_DATE=`date +%Y%m%d%H%M%S`
BAK_FILE_DATE=`date +%Y%m%d`
BAK_FILE_DIR="/usr/local/mysqldata/bak/$BAK_FILE_DATE"
# BAK_FILE_DIR="/usr/local/mysqldata/bak/$(date +%Y%m%d)"

# shell script
echo start $(date +%Y-%m-%d_%H:%M:%S).

mkdir -p $BAK_FILE_DIR

echo export start.
tabs="tb1 tb2"
echo export tbs...
$EXPORT_CMD_PREFIX $tbs > $BAK_FILE_DIR/tbs.sql
echo export tbs...done.
echo export end.

echo import start.
echo import tbs...
IMPORT_CMD_PREFIX < $BAK_FILE_DIR/tbs.sql
echo import tbs...done.
echo import end.

echo end $(date +%Y-%m-%d_%H:%M:%S).
```

### 1.2 脚本文件添加可执行权限

```bash
chmod u+x /usr/local/mysqldata/bak/bak.sh
```

### 1.3 测试脚本文件

```bash
sh /usr/local/mysqldata/bak/bak.sh
# 或者
cd /usr/local/mysqldata/bak
./bak.sh
```

## 2 添加计划任务 - crontab

```bash
# 编辑任务。会打开 vi，增加上述脚本文件的配置，下面是每1分钟执行一次。实际会对应到 /var/spool/cron 目录下的 root （当前用户）文件中。
crontab -e
*/1 * * * * /usr/local/mysqldata/bak/bak.sh

# 查看任务
crontab -l
*/1 * * * * /usr/local/mysqldata/bak/bak.sh。

# 如果任务执行失败了，可以通过以下命令查看任务日志
tail -f /var/log/cron

# 输出类似如下：
Oct 31 15:19:18 gridserver crontab[8951]: (root) REPLACE (root)
Oct 31 15:19:18 gridserver crontab[8951]: (root) END EDIT (root)
Oct 31 15:20:01 gridserver crond[4487]: (root) RELOAD (/var/spool/cron/root)
Oct 31 15:20:01 gridserver CROND[8965]: (root) CMD (/usr/lib64/sa/sa1 1 1)
Oct 31 15:20:01 gridserver CROND[8966]: (root) CMD (/usr/local/mysqldata/bak/bak.sh)
Oct 31 15:21:01 gridserver CROND[8980]: (root) CMD (/usr/local/mysqldata/bak/bak.sh)
Oct 31 15:22:01 gridserver CROND[8991]: (root) CMD (/usr/local/mysqldata/bak/bak.sh)
Oct 31 15:23:01 gridserver CROND[9001]: (root) CMD (/usr/local/mysqldata/bak/bak.sh)
Oct 31 15:24:01 gridserver CROND[9015]: (root) CMD (/usr/local/mysqldata/bak/bak.sh)
```

---

- [MySQL 教程合集](https://mp.weixin.qq.com/s/jflrWU62pBtevS62lEIHkQ)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)