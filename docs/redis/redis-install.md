---
title: Redis 安装
date: 2020-05-26 12:16:23
categories: Redis
tags: [Redis, 安装]
toc: true
---
手把手带你使用多种姿势安装 Redis 。
<!-- more -->

## 1 简介

Redis(Remote Dictionary Service) 是一个使用 ANSI C 编写的开源、支持网络、基于内存、可选持久性的键值对存储数据库。从 2015 年 6 月开始，Redis 的开发由 Redis Labs 赞助，而 2013 年 5 月至 2015 年 6 月期间，其开发由 Pivotal 赞助。在 2013 年 5 月之前，其开发由 VMware 赞助。根据月度排行网站 [DB-Engines.com](https://db-engines.com) 的数据显示，Redis 是最流行的键值对存储数据库。

很多人想到 Redis ，就想到缓存。但实际上 Redis 除了缓存之外，还有许多更加丰富的使用场景，比如分布式锁、限流等。

主要有如下特点：

- Redis 支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用，不会造成数据丢失。
- Redis 支持多种不同的数据结构类型之间的映射，包括简单的 key/value 类型的数据，同时还提供 list，set，zset，hash 等数据结构的存储。
- Redis 支持 master-slave 模式的数据备份。

主要有如下功能：

- 内存存储和持久化：redis 支持异步将内存中的数据写到硬盘上，在持久化的同时不影响继续服务。
- 取最新 N 个数据的操作，如：可以将最新的 10 条评论的 ID 放在 Redis 的 List 集合里面。
- 数据可以设置过期时间。
- 自带发布、订阅消息系统。
- 定时器、计数器。

## 2 安装
 
主要有 4 种方式获取 Redis ：

1. 编译安装（推荐）。
2. 使用 Docker 安装。
3. 直接安装。
4. 在线体验。

### 2.1 编译安装（推荐）

```bash
# 准备 gcc 环境
yum install gcc-c++

# 下载并安装
cd /usr/local
wget http://download.redis.io/releases/redis-5.0.8.tar.gz
tar -zxvf redis-5.0.8.tar.gz
cd redis-5.0.8/
make
make install
```

### 2.2 使用 Docker 安装

Docker 安装好之后，启动 Docker ，直接运行安装命令即可：

```bash
docker run --name redis -d -p 6379:6379 redis --requirepass 123456
```

Docker 上的 Redis 启动成功之后，可以从宿主机上连接（前提是宿主机上存在 redis-cli）：

```bash
redis-cli -a 123456
```

如果宿主机上没有安装 Redis ，那么也可以进入到 Docker 容器中去操作 Redis:

```bash
docker exec -it redis redis-cli -a 123456
```

### 2.3 直接安装

- CentOS：`yum install redis`
- Ubuntu：`apt-get install redis`
- Mac：`brew install redis`

### 2.4 在线体验

在线体验地址：[http://try.redis.io/](http://try.redis.io/) 

## 3 启动

首先，修改 `redis.conf` 配置文件，将里面的 `daemonize no` 改成 `daemonize yes`，让服务在后台启动：

```bash
vi /usr/local/redis-5.0.8/redis.conf

daemonize yes
```

然后，通过 `redis-server redis.conf` 命令启动 Redis ，如下：

```bash
cd /usr/local/redis-5.0.8

redis-server redis.conf
```

## 4 连接

通过 `redis-cli` 命令进入到控制台，然后通过 `ping` 命令进行连通性测试，如果看到 `pong` ，表示连接成功了，如下：

```bash
cd /usr/local/redis-5.0.8

redis-cli

# 127.0.0.1:6379> ping
# PONG
```

如果在 `redis.conf` 配置文件配置了密码 `requirepass 123456` ，则可通过 `redis-cli -a 123456` 连接。

---

也可以使用可视化工具来连接 Redis ，比如：`Redis Desktop Manager` 。

## 5 关闭

通过 `shutdown` 命令关闭 Redis ，如下：

```bash
cd /usr/local/redis-5.0.8

redis-cli

# 127.0.0.1:6379> shutdown
# not connected> exit
```

---

- [Redis 教程合集](https://mp.weixin.qq.com/s/iivXrj1cfTiPy89ueE_53Q)（微信左下方**阅读全文**可直达）。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)