---
title: Redis 实现分布式锁
date: 2020-06-26 16:15:32
categories: Redis
tags: [Redis]
toc: true
---
本文学习在 Redis 中通过 `String` 实现分布式锁。
<!-- more -->

## 1 概述

> 业务场景：一个简单的用户操作，一个线程去修改用户的状态，首先从数据库中读出用户的状态，然后在内存中进行修改，修改完成后，再存回去。在单线程中，这个操作没有问题，但是在多线程中，由于读取、修改、存这是三个操作，不是原子操作，这样会出问题。

对于这种类似问题，我们可以使用分布式锁来限制程序的并发执行。分布式锁实现的思路很简单，就是进来一个线程先占位，当别的线程进来操作时，发现已经有人占位了，就会放弃或者稍后再试。

## 2 基本使用

Redis 作为分布式锁，我们可以使用 `String` 数据结构来实现，通过 `setnx` 命令来占位，先进来的线程先占位，线程的操作执行完成后，再调用 `del` 指令释放位置。

```java
@Test
public void testDistributedLock() {
      Redis redis = new Redis();
      redis.execute(jedis -> {
      Long setnx = jedis.setnx("k1", "v1");
      if (setnx == 1) {
            // 没人占位
            jedis.set("name", "cxy35");
            String name = jedis.get("name");
            System.out.println(name);
            jedis.del("k1");// 释放资源
      } else {
            // 有人占位，停止/暂缓操作
      }
      });
}
```

上面的代码存在一个问题：如果代码业务执行的过程中抛异常或者挂了，这样会导致 `del` 指令没有被调用，这样，`k1` 无法释放，后面来的请求全部堵塞在这里，锁也永远得不到释放。

要解决这个问题，我们可以给锁添加一个过期时间，确保锁在一定的时间之后，能够得到释放。改进后的代码如下：

```java
@Test
public void testDistributedLock2() {
      Redis redis = new Redis();
      redis.execute(jedis -> {
      Long setnx = jedis.setnx("k1", "v1");
      if (setnx == 1) {
            // 给锁添加一个过期时间，防止应用在运行过程中抛出异常导致锁无法及时得到释放
            jedis.expire("k1", 5);
            // 没人占位
            jedis.set("name", "cxy35");
            String name = jedis.get("name");
            System.out.println(name);
            jedis.del("k1");// 释放资源
      } else {
            // 有人占位，停止/暂缓操作
      }
      });
}
```

这样改造之后，还有一个问题，就是在获取锁和设置过期时间之间，如果服务器突然挂掉了，这个时候锁被占用，无法及时得到释放，也会造成死锁，因为获取锁和设置过期时间是两个操作，不具备原子性。

为了解决这个问题，从 Redis 2.8 开始， `setnx` 和 `expire` 可以通过一个命令一起来执行了，我们对上述代码再做改进：

```java
@Test
public void testDistributedLock3() {
      Redis redis = new Redis();
      redis.execute(jedis -> {
      String set = jedis.set("k1", "v1", new SetParams().nx().ex(5));
      if ("OK".equals(set)) {
            // 没人占位
            jedis.set("name", "cxy35");
            String name = jedis.get("name");
            System.out.println(name);
            jedis.del("k1");// 释放资源
      } else {
            // 有人占位，停止/暂缓操作
      }
      });
}
```

## 3 使用 Lua 脚本解决超时问题

在上述代码中，为了防止业务代码在执行的时候抛出异常，我们给每一个锁添加了一个超时时间，超时之后，锁会被自动释放，但是这也带来了一个新的问题：如果要执行的业务非常耗时，可能会出现紊乱。举个例子：第一个线程首先获取到锁，然后开始执行业务代码，但是业务代码比较耗时，执行了 8 秒，这样，会在第一个线程的任务还未执行成功锁就会被释放了，此时第二个线程会获取到锁开始执行，在第二个线程刚执行了 3 秒，第一个线程也执行完了，此时第一个线程会释放锁，但是注意，它释放的第二个线程的锁，释放之后，第三个线程进来。

对于这个问题，我们可以从两个角度入手：

1. 尽量避免在获取锁之后，执行耗时操作。
2. 可以在锁上面做文章，将锁的 value 设置为一个随机字符串，每次释放锁的时候，都去比较随机字符串是否一致，如果一致，再去释放，否则，不释放。

对于第二种方案，由于释放锁的时候，要去查看锁的 `value`，第二步比较 `value` 的值是否正确，第三步释放锁，有三个步骤，很明显三个步骤不具备原子性，为了解决这个问题，我们得引入 `Lua` 脚本。

Lua 脚本的优势：

1. 使用方便，`Redis` 中内置了对 `Lua` 脚本的支持。
2. **`Lua` 脚本可以在 `Redis` 服务端原子的执行多个 `Redis` 命令。**

由于网络在很大程度上会影响到 `Redis` 性能，而使用 `Lua` 脚本可以让多个命令一次执行，可以有效解决网络给 `Redis` 带来的性能问题。在 `Redis` 中，使用 `Lua` 脚本，大致上有两种思路：

1. 提前在 `Redis` 服务端写好 `Lua` 脚本，然后在 `Java` 客户端去调用脚本（**推荐**）。
2. 可以直接在 `Java` 端去写 `Lua` 脚本，写好之后，需要执行时，每次将脚本发送到 `Redis` 上去执行。

首先在 `Redis` 服务端创建 `Lua` 脚本，内容如下：

```bash
cd /usr/local/redis-5.0.8
mkdir lua
vi releasewherevalequal.lua

if redis.call("get", KEYS[1]) == ARGV[1] then
   return redis.call("del", KEYS[1])
else
   return 0
end
```

接下来，可以给 `Lua` 脚本求一个 `SHA1` 和，命令如下：

```bash
cat lua/releasewherevalueequal.lua | redis-cli -a 123456 script load --pipe
"b8059ba43af6ffe8bed3db65bac35d452f8115d8"
```

上述 `script load` 会在 `Redis` 服务器中缓存 `Lua` 脚本，并返回脚本内容的 `SHA1` 校验和，然后在 `Java` 端调用时，传入 `SHA1` 校验和作为参数，这样 `Redis` 服务端就知道执行哪个脚本了。

---

接下来，在 `Java` 端调用这个脚本。

```java
@Test
public void testDistributedLock4ByLua() {
      Redis redis = new Redis();
      for (int i = 0; i < 2; i++) {
      redis.execute(jedis -> {
            // 1.先获取一个随机字符串
            String value = UUID.randomUUID().toString();
            // 2.获取锁
            String set = jedis.set("k1", value, new SetParams().nx().ex(5));
            // 3.判断是否成功拿到锁
            if ("OK".equals(set)) {
                  // 4.具体的业务操作
                  jedis.set("name", "cxy35");
                  String name = jedis.get("name");
                  System.out.println(name);
                  // 5.调用对应的 Lua 脚本释放锁
                  jedis.evalsha("b8059ba43af6ffe8bed3db65bac35d452f8115d8", Arrays.asList("k1"), Arrays.asList(value));
            } else {
                  System.out.println("没拿到锁");
            }
      });
      }
}
```

执行结果如下：

```
cxy35
没拿到锁
```

---

- [Redis 教程合集](https://mp.weixin.qq.com/s/iivXrj1cfTiPy89ueE_53Q)（微信左下方**阅读全文**可直达）。
- 本文示例代码：[https://github.com/cxy35/samples/tree/master/redis/redis-jedis](https://github.com/cxy35/samples/tree/master/redis/redis-jedis)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)