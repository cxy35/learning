---
title: Redis 常用命令
date: 2020-06-01 14:52:53
categories: Redis
tags: [Redis, 命令]
toc: true
---
通过本文学习 Redis 常用命令。
<!-- more -->

## 1 基本数据类型相关命令

参考 [Redis 基本数据类型（字符串、列表、集合、散列、有序集合）](https://mp.weixin.qq.com/s/BdHrMOU_9UwUNYDmMMrD6w) 。

## 2 key 相关命令

key 相关的命令，对不同的数据类型都通用。

- del

以删除一个已经存在的 key 。

```bash
127.0.0.1:6379> set k1 hello
OK
127.0.0.1:6379> del k1
(integer) 1
127.0.0.1:6379> get k1
(nil)
```

- exists

检测一个给定的 key 是否存在。

```bash
127.0.0.1:6379> set k1 hello
OK
127.0.0.1:6379> exists k1
(integer) 1
127.0.0.1:6379> exists k2
(integer) 0
```

- keys

获取满足给定模式的所有 key 。 `keys *` 表示获取所有的 key ， `*` 也可以是一个正则表达式。

```bash
127.0.0.1:6379> mset k1 hello k2 redis name cxy35
OK
127.0.0.1:6379> keys *
1) "name"
2) "k2"
3) "k1"
127.0.0.1:6379> keys k*
1) "k2"
2) "k1"
```

- ttl/pttl

查看 key 的有效期（秒/毫秒）。 -2 表示 key 不存在或者已过期， -1 表示 key 存在并且没有设置过期时间（永久有效）。默认为 -1 。

```bash
127.0.0.1:6379> set k1 hello
OK
127.0.0.1:6379> ttl k1
(integer) -1
127.0.0.1:6379> ttl k2
(integer) -2
```

- expire/pexpire

给 key 设置有效期（秒/毫秒），在有效期过后，key 会被销毁。

```bash
127.0.0.1:6379> set k1 hello
OK
127.0.0.1:6379> ttl k1
(integer) -1
127.0.0.1:6379> expire k1 10
(integer) 1
127.0.0.1:6379> ttl k1
(integer) 6
```

- persist

移除一个 key 的过期时间，这样该 key 就永远不会过期。

```bash
127.0.0.1:6379> set k1 hello
OK
127.0.0.1:6379> ttl k1
(integer) -1
127.0.0.1:6379> expire k1 30
(integer) 1
127.0.0.1:6379> ttl k1
(integer) 27
127.0.0.1:6379> persist k1
(integer) 1
127.0.0.1:6379> ttl k1
(integer) -1
```

- dump

序列化给定的 key，并返回序列化之后的值。

```bash
127.0.0.1:6379> set k1 hello
OK
127.0.0.1:6379> dump k1
"\x00\x05hello\t\x00\xb3\x80\x8e\xba1\xb2C\xbb"
```

---

- [Redis 教程合集](https://mp.weixin.qq.com/s/iivXrj1cfTiPy89ueE_53Q)（微信左下方**阅读全文**可直达）。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)