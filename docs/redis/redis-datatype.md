---
title: Redis 基本数据类型（字符串、列表、集合、散列、有序集合）
date: 2020-05-31 16:58:08
categories: Redis
tags: [Redis]
toc: true
---
Redis 中的数据都是以 `key/value` 的形式存储的，key 都是字符串，value 支持多种不同的数据类型，其中基本数据类型有：`String(含 Bit)、List、Set、Hash、ZSet`。
<!-- more -->

## 1 String

String 是 redis 中最基本的数据类型，redis 中的 String 类型是二进制安全的，即它可以包含任何数据，比如一个序列化的对象甚至一张 jpg 图片，要注意的是 redis 中的字符串大小上限是 512M 。

String 是动态字符串，内部可以修改，像 Java 中的 StringBuﬀer ，它采用分配冗余空间的方式来减少内存的频繁分配。在 Redis 内部结构中，一般实际分配的内存会大于需要的内存，当字符串小于 1M 的时候，扩容都是在现有的空间基础上加倍，扩容每次扩 1M 空间，最大 512M 。

### 1.1 String

- set

给一个 key 设置 value 。

```bash
127.0.0.1:6379> set k1 hello
OK
```

- get

获取对应 key 的 value，如果 key 不存在则返回 nil 。

```bash
127.0.0.1:6379> set k1 hello
OK
127.0.0.1:6379> get k1
"hello"
127.0.0.1:6379> get k2
(nil)
```

- getrange

用来返回 key 对应的 value 的子串，类似于 Java 里的 substring 。子串由 start 和 end 决定，从左往右计算，如果下标是负数，则从右往左计算，其中 -1 表示最后一个字符， -2 是倒数第二个...

```bash
127.0.0.1:6379> set k1 helloredis
OK
127.0.0.1:6379> getrange k1 0 4
"hello"
127.0.0.1:6379> getrange k1 0 -1
"helloredis"
127.0.0.1:6379> getrange k1 -5 -1
"redis"
```

- setrange

用来覆盖一个已经存在的 key 的 value 。

```bash
127.0.0.1:6379> set k1 helloredis
OK
127.0.0.1:6379> get k1
"helloredis"
127.0.0.1:6379> setrange k1 5 world
(integer) 10
127.0.0.1:6379> get k1
"helloworld"
```

- mset/mget

用来批量设置值和批量获取值。

```bash
127.0.0.1:6379> mset k1 java k2 php k3 c++
OK
127.0.0.1:6379> mget k1 k2 k3
1) "java"
2) "php"
3) "c++"
```

- getset

获取 key 所对应的 value ，并对 key 进行重置 。

```bash
127.0.0.1:6379> set k1 hello
OK
127.0.0.1:6379> get k1
"hello"
127.0.0.1:6379> getset k1 helloredis
"hello"
127.0.0.1:6379> get k1
"helloredis"
```

- append

使用 append 命令时，如果 key 已经存在，则直接在对应的 value 后追加值，否则就创建新的键值对。

```bash
127.0.0.1:6379> append k1 hello
(integer) 5
127.0.0.1:6379> get k1
"hello"
127.0.0.1:6379> append k1 redis
(integer) 10
127.0.0.1:6379> get k1
"helloredis"
```

- strlen

用来计算 key 的 value 的长度。

```bash
127.0.0.1:6379> set k1 helloredis
OK
127.0.0.1:6379> strlen k1
(integer) 10
```

- setnx

是 **set** if **n**ot e**x**ists 的简写，set 命令在执行时，如果 key 已经存在，则新值会覆盖掉旧值，而对于 setnx 命令，如果 key 已经存在，则不做任何操作，如果 key 不存在，则效果等同于 set 命令。

```bash
127.0.0.1:6379> set k1 helloredis
OK
127.0.0.1:6379> get k1
"helloredis"
127.0.0.1:6379> setnx k1 helloworld
(integer) 0
127.0.0.1:6379> get k1
"helloredis"
127.0.0.1:6379> setnx k2 helloworld
(integer) 1
127.0.0.1:6379> get k2
"helloworld"
```

- msetnx

兼具了 setnx 和 mset 的特性，但在执行时，如果有一个 key 存在，则所有的都不会执行。

```bash
127.0.0.1:6379> set k1 java
OK
127.0.0.1:6379> get k1
"java"
127.0.0.1:6379> msetnx k1 php k2 c++
(integer) 0
127.0.0.1:6379> get k1
"java"
127.0.0.1:6379> get k2
(nil)
```

- ttl/pttl

查看 key 的有效期（秒/毫秒）， -1 表示一直有效， -2 表示已过期，默认为 -1 。

- setex

用来给 key 设置 value ，同时设置过期时间（单位是秒），等效于先给 key 设置 value ，再给 key 设置过期时间。

```bash
127.0.0.1:6379> setex k1 10 helloredis
OK
127.0.0.1:6379> ttl k1
(integer) 8
127.0.0.1:6379> pttl k1
(integer) 3040
127.0.0.1:6379> get k1
(nil)
```

- psetex

作用和 setex 类似，不同的是，这里设置过期时间的单位是毫秒。

```bash
127.0.0.1:6379> psetex k1 10000 helloredis
OK
127.0.0.1:6379> ttl k1
(integer) 8
127.0.0.1:6379> pttl k1
(integer) 3575
```

- incr/decr

可以对指定 key 的 value 执行加/减 1 操作，如果指定的 key 不存在，那么在加/减 1 操作之前，会先将 key 的 value 设置为 0 ，如果 key 的 value 不是数字，则会报错。

```bash
127.0.0.1:6379> incr k1
(integer) 1
127.0.0.1:6379> incr k1
(integer) 2
```

- incrby/decrby

和 incr/decr 功能类似，不同的是可以指定步长。

```bash
127.0.0.1:6379> incrby k1 5
(integer) 5
127.0.0.1:6379> incrby k1 5
(integer) 10
127.0.0.1:6379> incrby k2 -2
(integer) -2
127.0.0.1:6379> incrby k2 -2
(integer) -4
```

- incrbyﬂoat

和 incrby 类似，但是自增的步长可以设置为浮点数。

```bash
127.0.0.1:6379> incrbyfloat k1 0.6
"0.6"
127.0.0.1:6379> incrbyfloat k1 0.6
"1.2"
127.0.0.1:6379> incrbyfloat k2 -0.8
"-0.8"
127.0.0.1:6379> incrbyfloat k2 -0.8
"-1.6"
```

### 1.2 Bit

在 Redis 中，字符串都是以二进制的方式来存储的。例如 `set k1 a` ， a 对应的 ASCII 码是 97 ，转为二进制是 `01100001` ，BIT 相关的命令就是对二进制进行操作的。

- getbit

可以返回 key 对应的 value 在 offset 处的 bit 值，以上文提到的 k1 为例， a 对应的二进制数据是 01100001 ，所以当 offset 为 0 时，对应的 bit 值为 0 ； offset 为 1 时，对应的 bit 值为 1 ； offset 为 2 时，对应的 bit 值为 1 ；offset 为 3 时，对应的 bit 值为 0，依此类推…

```bash
127.0.0.1:6379> set k1 a
OK
127.0.0.1:6379> getbit k1 0
(integer) 0
127.0.0.1:6379> getbit k1 1
(integer) 1
127.0.0.1:6379> getbit k1 2
(integer) 1
127.0.0.1:6379> getbit k1 3
(integer) 0
127.0.0.1:6379> getbit k1 4
(integer) 0
127.0.0.1:6379> getbit k1 5
(integer) 0
127.0.0.1:6379> getbit k1 6
(integer) 0
127.0.0.1:6379> getbit k1 7
(integer) 1
```

- setbit

可以用来修改二进制数据，比如 a 对应的 ASCII 码为 97，c 对应的 ASCII 码为 99，97 转为二进制是 01100001 ，99 转为二进制是 01100011 ，两个的差异在于第六位一个是 0 一个是 1 ，通过 SETBIT 命令，我们可以将 k1 的第六位的 0 改为 1 （第六位是从 0 开始算）。

```bash
127.0.0.1:6379> set k1 a
OK
127.0.0.1:6379> setbit k1 6 1
(integer) 0
127.0.0.1:6379> get k1
"c"
```

此时，k1 中存储的字符也就变为了 c 。 setbit 在执行时所返回的数字，表示该位上原本的 bit 值。

- bitcount

可以用来统计这个二进制数据中 1 的个数。

```bash
127.0.0.1:6379> set k1 a
OK
127.0.0.1:6379> bitcount k1
(integer) 3
```

关于 bitcount ， redis 官网上有一个非常有意思的案例：用户上线次数统计。节选部分原文如下：

> 举个例子，如果今天是网站上线的第 100 天，而用户 peter 在今天阅览过网站，那么执行命令 setbit peter 100 1 ；如果明天 peter 也继续阅览网站，那么执行命令 setbit peter 101 1 ，以此类推。当要计算 peter 总共以来的上线次数时，就使用 bitcount 命令：执行 bitcount peter ，得出的结果就是 peter 上线的总天数。

这种统计方式最大的好处就是节省空间并且运算速度快。每天占用一个 bit，一年也就 365 个 bit，10 年也就 10*365 个 bit ，也就是 456 个字节，对于这么大的数据，bit 的操作速度非常快。

- bitop

可以对一个或者多个二进制位串执行并 (and)、或 (or)、异或 (xor) 以及非 (not) 运算，如下：a 对应的 ASCII 码转为二进制是 `01100001` ，c 对应的二进制位串是 `01100011` 。对这两个二进制位串分别执行 and\or\xor 的结果如下：

```bash
127.0.0.1:6379> set k1 a
OK
127.0.0.1:6379> set k2 c
OK
127.0.0.1:6379> bitop and k3 k1 k2
(integer) 1
127.0.0.1:6379> get k3
"a"
127.0.0.1:6379> bitop or k3 k1 k2
(integer) 1
127.0.0.1:6379> get k3
"c"
127.0.0.1:6379> bitop xor k3 k1 k2
(integer) 1
127.0.0.1:6379> get k3
"\x02"
```

另外， bitop 也可以执行 not 运算，但是注意参数个数，如下：

```bash
127.0.0.1:6379> bitop not k3 k4
(integer) 1
```

这里会对 k4 的二进制位串取反，将取反结果交给 k3 。

- bitpos

用来获取二进制位串中第一个 1 或者 0 的位置。

```bash
127.0.0.1:6379> set k1 a
OK
127.0.0.1:6379> bitpos k1 1
(integer) 1
127.0.0.1:6379> bitpos k1 0
(integer) 0
```

也可以在后面设置一个范围，不过后面的范围是字节的范围，而不是二进制位串的范围。

## 2 List

List 是一个简单的**字符串列表，按照插入顺序进行排序**，我们可以从 List 的头部 (LEFT) 或者尾部 (RIGHT) 插入或弹出一个元素。

- lpush/rpush

将一个或多个值 value 插入到列表 key 的表头，如果有多个 value 值，那么各个 value 值按从左到右/从右到左的顺序依次插入到表头，类似入栈。如果 key 不存在，那么在进行 push 操作前会创建一个空列表。 如果 key 对应的值不是一个 list 的话，那么会返回一个错误。

```bash
127.0.0.1:6379> lpush k1 aa bb cc
(integer) 3
127.0.0.1:6379> lrange k1 0 -1
1) "cc"
2) "bb"
3) "aa"
127.0.0.1:6379> rpush k2 aa bb cc
(integer) 3
127.0.0.1:6379> lrange k2 0 -1
1) "aa"
2) "bb"
3) "cc"
```

- lrange

返回列表 key 中指定区间内的元素，区间以偏移量 start 和 stop 指定，下标 (index) 参数 start 和 stop 都以 0 为底，即 0 表示列表的第一个元素，1 表示列表的第二个元素，以此类推。我们也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。

```bash
127.0.0.1:6379> lpush k1 aa bb cc
(integer) 3
127.0.0.1:6379> lrange k1 0 1
1) "cc"
2) "bb"
127.0.0.1:6379> lrange k1 0 -1
1) "cc"
2) "bb"
3) "aa"
```

- lindex

可以返回列表 key 中，下标为 index 的元素，正数下标 0 表示第一个元素，也可以使用负数下标，-1 表示倒数第一个元素。

```bash
127.0.0.1:6379> lpush k1 aa bb cc
(integer) 3
127.0.0.1:6379> lrange k1 0 -1
1) "cc"
2) "bb"
3) "aa"
127.0.0.1:6379> lindex k1 0
"cc"
127.0.0.1:6379> lindex k1 -1
"aa"
```

- ltrim

可以对一个列表进行修剪，即让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。下标与之前介绍的写法都一致，这里不赘述。

```bash
127.0.0.1:6379> lpush k1 aa bb cc
(integer) 3
127.0.0.1:6379> lrange k1 0 -1
1) "cc"
2) "bb"
3) "aa"
127.0.0.1:6379> ltrim k1 0 1
OK
127.0.0.1:6379> lrange k1 0 -1
1) "cc"
2) "bb"
```

- lpop/rpop

可以移除并返回列表 key 的头/尾元素。

```bash
127.0.0.1:6379> lpush k1 aa bb cc
(integer) 3
127.0.0.1:6379> lrange k1 0 -1
1) "cc"
2) "bb"
3) "aa"
127.0.0.1:6379> lpop k1
"cc"
127.0.0.1:6379> lrange k1 0 1
1) "bb"
2) "aa"
127.0.0.1:6379> rpop k1
"aa"
127.0.0.1:6379> lrange k1 0 1
1) "bb"
```

- blpop/brpop

阻塞式列表的弹出原语。它是命令 lpop/rpop 的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被该命令阻塞。当给定多个 key 参数时，按参数 key 的先后顺序依次检查各个列表，弹出第一个非空列表的头元素。同时，在使用该命令时也需要指定阻塞的时长，时长单位为秒，在该时长内如果没有元素可供弹出，则阻塞结束。返回的结果是 key 和 value 的组合。

```bash
127.0.0.1:6379> lpush k1 aa bb cc
(integer) 3
127.0.0.1:6379> lrange k1 0 -1
1) "cc"
2) "bb"
3) "aa"
127.0.0.1:6379> blpop k1 10
1) "k1"
2) "cc"
127.0.0.1:6379> blpop k1 10
1) "k1"
2) "bb"
127.0.0.1:6379> blpop k1 10
1) "k1"
2) "aa"
127.0.0.1:6379> blpop k1 10
(nil)
(10.06s)
```

## 3 Set

Set 是 **String 类型的无序集合**，不同于 List ，Set 中的元素**不可以重复**。

- sadd

添加一个或多个指定的 member 元素到集合的 key 中，指定的一个或者多个元素 member 如果已经在集合 key 中存在则忽略，如果集合 key 不存在，则新建集合 key ，并添加 member 元素到集合 key 中。

```bash
127.0.0.1:6379> sadd k1 aa bb cc
(integer) 3
```

- smembers

返回 key 集合所有的元素。

```bash
127.0.0.1:6379> sadd k1 aa bb cc
(integer) 3
127.0.0.1:6379> smembers k1
1) "cc"
2) "bb"
3) "aa"
```

- srem

在 key 集合中移除指定的元素，如果指定的元素不是 key 集合中的元素则忽略。如果 key 集合不存在则被视为一个空的集合，该命令返回 0 。

```bash
127.0.0.1:6379> sadd k1 aa bb cc
(integer) 3
127.0.0.1:6379> srem k1 aa
(integer) 1
127.0.0.1:6379> srem k1 dd
(integer) 0
127.0.0.1:6379> smembers k1
1) "cc"
2) "bb"
```

- sismember

返回成员 member 是否是存储的集合 key 的成员。

```bash
127.0.0.1:6379> sadd k1 aa bb cc
(integer) 3
127.0.0.1:6379> sismember k1 aa
(integer) 1
127.0.0.1:6379> sismember k1 dd
(integer) 0
```

- scard

返回 key 对应的集合中成员的数量。

```bash
127.0.0.1:6379> sadd k1 aa bb cc
(integer) 3
127.0.0.1:6379> scard k1
(integer) 3
```

- srandmember

仅需我们提供 key 参数，它就会随机返回 key 集合中的一个元素，从 Redis 2.6 开始，该命令也可以接受一个可选的 count 参数，如果 count 是整数且小于元素的个数，则返回 count 个随机元素，如果 count 是整数且大于集合中元素的个数时，则返回集合中的所有元素，当 count 是负数，则会返回一个包含 count 的绝对值的个数元素的数组，如果 count 的绝对值大于元素的个数，则返回的结果集里会出现一个元素出现多次的情况。

```bash
127.0.0.1:6379> sadd k1 aa bb cc
(integer) 3
127.0.0.1:6379> srandmember k1
"aa"
127.0.0.1:6379> srandmember k1 2
1) "aa"
2) "bb"
127.0.0.1:6379> srandmember k1 5
1) "cc"
2) "aa"
3) "bb"
127.0.0.1:6379> srandmember k1 -1
1) "cc"
127.0.0.1:6379> srandmember k1 -5
1) "cc"
2) "aa"
3) "aa"
4) "aa"
5) "cc"
```

- spop

用法和 srandmember 类似，不同的是， spop 每次选择一个随机的元素之后，该元素会出栈，而 srandmember 则不会出栈，只是将该元素展示出来。

```bash
127.0.0.1:6379> sadd k1 aa bb cc
(integer) 3
127.0.0.1:6379> spop k1
"cc"
127.0.0.1:6379> smembers k1
1) "bb"
2) "aa"
```

- smove

将 source 集合中的一个 member 从 source 集合移动到 destination 集合中。

```bash
127.0.0.1:6379> sadd k1 aa bb cc
(integer) 3
127.0.0.1:6379> smove k1 k2 aa
(integer) 1
127.0.0.1:6379> smembers k1
1) "cc"
2) "bb"
127.0.0.1:6379> smembers k2
1) "aa"
```

- sdiﬀ/sinter/sunion

用来返回一个集合与给定集合的差集/交集/并集的元素。

```bash
127.0.0.1:6379> sadd k1 aa bb cc
(integer) 3
127.0.0.1:6379> sadd k2 bb cc dd
(integer) 3
127.0.0.1:6379> sdiff k1 k2
1) "aa"
127.0.0.1:6379> sdiff k2 k1
1) "dd"
127.0.0.1:6379> sinter k1 k2
1) "cc"
2) "bb"
127.0.0.1:6379> sunion k1 k2
1) "aa"
2) "bb"
3) "dd"
4) "cc"
```

- sdiﬀstore/sinterstore/sunionstore

与 sdiﬀ/sinter/sunion 命令基本一致，不同的是会将结果保存在一个集合中。

```bash
127.0.0.1:6379> sadd k1 aa bb cc
(integer) 3
127.0.0.1:6379> sadd k2 bb cc dd
(integer) 3
127.0.0.1:6379> sdiffstore k3 k1 k2
(integer) 1
127.0.0.1:6379> smembers k3
1) "aa"
127.0.0.1:6379> sinterstore k4 k1 k2
(integer) 2
127.0.0.1:6379> smembers k4
1) "cc"
2) "bb"
127.0.0.1:6379> sunionstore k5 k1 k2
(integer) 4
127.0.0.1:6379> smembers k5
1) "aa"
2) "bb"
3) "dd"
4) "cc"
```

## 4 Hash

Hash 类似于 Java 中的 `Map` ，是一个**键值对集合**，在 Redis 中可以用来存储对象。很多时候， Hash 就像一个微缩版的 Redis 。

在 Hash 结构中， key 是一个字符串， value 则是一个 key/value 键值对。

- hset

用来设置 key 指定的哈希集中指定字段的值。

```bash
127.0.0.1:6379> hset k1 name cxy35
(integer) 1
```

- hget

用来返回 key 指定的哈希集中该字段所关联的值。

```bash
127.0.0.1:6379> hset k1 name cxy35
(integer) 1
127.0.0.1:6379> hget k1 name
"cxy35"
```

- hmset/hmget

批量设置/返回 key 指定的哈希集中指定字段的值。

```bash
127.0.0.1:6379> hmset k1 name cxy35 age 18 city HZ
OK
127.0.0.1:6379> hmget k1 name age city
1) "cxy35"
2) "18"
3) "HZ"
```

- hvals

返回 key 指定的哈希集中所有字段的值。

```bash
127.0.0.1:6379> hmset k1 name cxy35 age 18 city HZ
OK
127.0.0.1:6379> hvals k1
1) "cxy35"
2) "18"
3) "HZ"
```

- hkeys

返回 key 指定的哈希集中所有字段的名字

```bash
127.0.0.1:6379> hmset k1 name cxy35 age 18 city HZ
OK
127.0.0.1:6379> hkeys k1
1) "name"
2) "age"
3) "city"
```

- hgetall

返回 key 指定的哈希集中所有的字段和值。返回值中，每个字段名的下一个是它的值，所以返回值的长度是哈希集大小的两倍。

```bash
127.0.0.1:6379> hmset k1 name cxy35 age 18 city HZ
OK
127.0.0.1:6379> hgetall k1
1) "name"
2) "cxy35"
3) "age"
4) "18"
5) "city"
6) "HZ"
```

- hdel

从 key 指定的哈希集中移除指定的域 ﬁeld ，在哈希集中不存在的域将被忽略。

```bash
127.0.0.1:6379> hmset k1 name cxy35 age 18 city HZ
OK
127.0.0.1:6379> hmget k1 name age city
1) "cxy35"
2) "18"
3) "HZ"
127.0.0.1:6379> hdel k1 city
(integer) 1
127.0.0.1:6379> hmget k1 name age city
1) "cxy35"
2) "18"
3) (nil)
```

- hsetnx

只在 key 指定的哈希集中不存在指定的字段时，设置字段的值，如果字段已存在，该操作无效果。

```bash
127.0.0.1:6379> hmset k1 name cxy35 age 18
OK
127.0.0.1:6379> hmget k1 name age
1) "cxy35"
2) "18"
127.0.0.1:6379> hsetnx k1 age 20
(integer) 0
127.0.0.1:6379> hsetnx k1 city HZ
(integer) 1
127.0.0.1:6379> hmget k1 name age city
1) "cxy35"
2) "18"
3) "HZ"
```

- hexists

返回 hash 里面 field 是否存在。

```bash
127.0.0.1:6379> hmset k1 name cxy35 age 18
OK
127.0.0.1:6379> hexists k1 name
(integer) 1
127.0.0.1:6379> hexists k1 city
(integer) 0
```

- hincrby

增加 key 指定的哈希集中指定字段的数值。如果 key 不存在，会创建一个新的哈希集并与 key 关联。如果字段不存在，则字段的值在该操作执行前被设置为 0 ， hincrby 支持的值的范围限定在 64 位有符号整数。

```bash
127.0.0.1:6379> hmset k1 name cxy35 age 18
OK
127.0.0.1:6379> hincrby k1 age 5
(integer) 23
127.0.0.1:6379> hget k1 age
"23"
127.0.0.1:6379> hincrby k2 age 3
(integer) 3
127.0.0.1:6379> hincrby k2 age 3
(integer) 6
127.0.0.1:6379> hget k2 age
"6"
```

- hincrbyﬂoat

与 hincrby 用法基本一致，只不过这里允许 float 类型的数据，不赘述。

```bash
127.0.0.1:6379> hincrbyfloat k2 age 2.5
"8.5"
```

- hlen

返回 key 指定的哈希集包含的字段的数量。

```bash
127.0.0.1:6379> hmset k1 name cxy35 age 18
OK
127.0.0.1:6379> hlen k1
(integer) 2
```

- hstrlen

返回 hash 指定 field 的 value 的字符串长度，如果 hash 或者 field 不存在，返回 0 。

```bash
127.0.0.1:6379> hmset k1 name cxy35 age 18
OK
127.0.0.1:6379> hstrlen k1 name
(integer) 5
127.0.0.1:6379> hstrlen k1 city
(integer) 0
```

## 5 ZSet

ZSet 和 Set 一样，也是 **String 类型的有序集合**，不同的是 ZSet 中的每个元素都会关联一个 double 类型的分数 score ，里面的元素总是通过 score 进行着排序。 ZSet 中的**成员都是唯一的**，但是所关联的分数可以重复。

- zadd

将所有指定成员添加到键为 key 的有序集合里面。添加时可以指定多个分数/成员（score/member）对。 如果指定添加的成员已经是有序集合里面的成员，则会更新该成员的分数（scrore）并更新到正确的排序位置。

```bash
127.0.0.1:6379> zadd k1 10 v1
(integer) 1
```

- zscore

返回有序集 key 中，成员 member 的 score 值。

```bash
127.0.0.1:6379> zadd k1 10 v1
(integer) 1
127.0.0.1:6379> zscore k1 v1
"10"
```

- zrange/zrevrange

根据 index 返回 member ，该命令在执行时加上 withscores 参数可以连同 score 一起返回。 zrevrange 与 zrange 类似，但是倒叙。

```bash
127.0.0.1:6379> zadd k1 10 v1 20 v2 30 v3
(integer) 3
127.0.0.1:6379> zrange k1 0 1
1) "v1"
2) "v2"
127.0.0.1:6379> zrevrange k1 0 1
1) "v3"
2) "v2"
127.0.0.1:6379> zrange k1 0 1 withscores
1) "v1"
2) "10"
3) "v2"
4) "20"
127.0.0.1:6379> zrange k1 0 -1 withscores
1) "v1"
2) "10"
3) "v2"
4) "20"
5) "v3"
6) "30"
```

- zrangebyscore

按照 score 范围返回 member ，加上 withscores 可以连 score 一起返回。

```bash
127.0.0.1:6379> zadd k1 10 v1 20 v2 30 v3
(integer) 3
127.0.0.1:6379> zrangebyscore k1 10 20
1) "v1"
2) "v2"
127.0.0.1:6379> zrangebyscore k1 10 20 withscores
1) "v1"
2) "10"
3) "v2"
4) "20"
```

- zrangebylex

返回有序集合中指定成员之间的成员。

```bash
127.0.0.1:6379> zadd k1 10 v1 20 v2 30 v3
(integer) 3
127.0.0.1:6379> zrangebylex k1 - +
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> zrangebylex k1 [v1 [v2
1) "v1"
2) "v2"
```

**注意：可以用 - 和 + 表示得分最小值和最大值，如果使用成员名的话，一定要在成员名之前加上 [ 。**

- zrem

从集合中弹出一个元素。

```bash
127.0.0.1:6379> zadd k1 10 v1 20 v2 30 v3
(integer) 3
127.0.0.1:6379> zrem k1 v1
(integer) 1
127.0.0.1:6379> zrange k1 0 -1 withscores
1) "v2"
2) "20"
3) "v3"
4) "30"
```

- zcard

返回 key 的有序集元素个数。

```bash
127.0.0.1:6379> zadd k1 10 v1 20 v2 30 v3
(integer) 3
127.0.0.1:6379> zcard k1
(integer) 3
```

- zcount

返回有序集 key 中， score 值在 min 和 max 之间（默认闭区间）的成员数量。

```bash
127.0.0.1:6379> zadd k1 10 v1 20 v2 30 v3
(integer) 3
127.0.0.1:6379> zcount k1 10 20
(integer) 2
127.0.0.1:6379> zcount k1 (10 20
(integer) 1
```

- zlexcount

返回有序集合中指定成员之间的成员数量。

```bash
127.0.0.1:6379> zadd k1 10 v1 20 v2 30 v3
(integer) 3
127.0.0.1:6379> zlexcount k1 - +
(integer) 3
127.0.0.1:6379> zlexcount k1 [v1 [v2
(integer) 2
```

**注意：可以用 - 和 + 表示得分最小值和最大值，如果使用成员名的话，一定要在成员名之前加上 [ 。**

- zrank/zrevrank

返回有序集 key 中成员 member 的排名。其中有序集成员按 score 值递增（从小到大/从大到小）顺序排列。排名以 0 为基数，即 score 值最小的成员排名为 0 。

```bash
127.0.0.1:6379> zadd k1 10 v1 20 v2 30 v3
(integer) 3
127.0.0.1:6379> zrank k1 v1
(integer) 0
127.0.0.1:6379> zrank k1 v3
(integer) 2
127.0.0.1:6379> zrevrank k1 v3
(integer) 0
```

- zincrby

为有序集 key 的成员 member 的 score 值加上增量 increment 。如果 key 中不存在 member ，就在 key 中添加一个 member ，score 是 increment（就好像它之前的 score 是0.0）。如果 key 不存在，就创建一个只含有指定 member 成员的有序集合。

```bash
127.0.0.1:6379> zadd k1 10 v1 20 v2 30 v3
(integer) 3
127.0.0.1:6379> zincrby k1 6 v1
"16"
127.0.0.1:6379> zscore k1 v1
"16"
127.0.0.1:6379> zincrby k2 3 v1
"3"
127.0.0.1:6379> zscore k2 v1
"3"
```

- zinterstore

计算给定的 numkeys 个有序集合的交集，并且把结果放到 destination 中。 在给定要计算的 key 和其它参数之前，必须先给定 key 个数 numberkeys 。该命令也可以在执行的过程中给原 score 乘以 weights 后再求和。

```bash
127.0.0.1:6379> zadd k2 2 v1
(integer) 1
127.0.0.1:6379> zadd k2 3 v2
(integer) 1
127.0.0.1:6379> zadd k2 4 v3
(integer) 1
127.0.0.1:6379> zadd k3 9 v2
(integer) 1
127.0.0.1:6379> zadd k3 10 v3
(integer) 1
127.0.0.1:6379> zadd k3 11 v4
(integer) 1
127.0.0.1:6379> zinterstore k4 2 k2 k3
(integer) 2
127.0.0.1:6379> zrange k4 0 -1 withscores
1) "v2"
2) "12"
3) "v3"
4) "14"
127.0.0.1:6379> zinterstore k5 2 k2 k3 weights 3 1
(integer) 2
127.0.0.1:6379> zrange k5 0 -1 withscores
1) "v2"
2) "18"
3) "v3"
4) "22"
```

---

- [Redis 教程合集](https://mp.weixin.qq.com/s/iivXrj1cfTiPy89ueE_53Q)（微信左下方**阅读全文**可直达）。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)