---
title: Redis 中的 Java 客户端（Jedis / Lettuce）
date: 2020-06-07 19:48:07
categories: Redis
tags: [Redis, Jedis, Lettuce]
toc: true
---
本文学习使用 Java 客户端（`Jedis / Lettuce`） 操作 Redis 。
<!-- more -->

## 1 开启远程连接

Redis 默认是不支持远程连接的，需要手动开启，修改 `redis.conf` ，主要有 3 个地方改动：

```bash
vi /usr/local/redis-5.0.8/redis.conf

# bind 127.0.0.1
protected-mode no
requirepass 123456
```

之后重新启动 Redis 。

## 2 Jedis

### 2.1 基本使用

Jedis 的 GitHub 地址：[https://github.com/xetorthio/jedis](https://github.com/xetorthio/jedis) 。

首先创建一个普通的 Maven 项目 `redis-jedis` ，项目创建成功后，添加 `Jedis` 依赖： 

```xml
<dependencies>
      <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.2.0</version>
            <type>jar</type>
            <scope>compile</scope>
      </dependency>
      <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13</version>
            <scope>test</scope>
      </dependency>
</dependencies>
```

然后创建一个测试类及测试方法：

```java
public class JedisTests {
    @Test
    public void testJedis() {
        // 1.构造一个 Jedis 对象，因为这里使用的默认端口 6379，所以不用配置端口
        Jedis jedis = new Jedis("192.168.71.62");
        // 2.密码认证
        jedis.auth("123456");

        // 3.测试是否连接成功
        String r = jedis.ping();
        // 4.返回  pong  表示连接成功
        System.out.println(r);

        jedis.set("k1", "hellojedis");
        String v = jedis.get("k1");
        System.out.println(v);
    }
}
```

在 Jedis 中，方法的 API 和 Redis 的命令高度一致，所以，Jedis 中的方法见名知意，直接使用即可。

### 2.2 连接池

因为 Jedis 对象不是线程安全的，所以在实际应用中，我们一般都是通过连接池来获取，使用完成之后，再还给连接池。

```java
@Test
public void testJedisPool() {
      Jedis jedis = null;
      // 1.构造一个 Jedis 连接池
      JedisPool pool = new JedisPool("192.168.71.62", 6379);
      // 2.从连接池中获取一个 Jedis 连接
      jedis = pool.getResource();
      jedis.auth("123456");
      try {
            // 3.Jedis 操作
            String r = jedis.ping();
            System.out.println(r);
      } catch (Exception e) {
            e.printStackTrace();
      } finally {
            // 4.归还连接
            if (jedis != null) {
                  jedis.close();
            }
      }
}
```

上述代码中，我们使用了 `try-catch-finally` 来确保 jedis 对象被关闭，也可以使用 JDK1.7 中的 `try-with-resources` 来实现（代码看着比之前简洁），改造如下：

```java
@Test
public void testJedisPool2() {
      JedisPool pool = new JedisPool("192.168.71.62");
      try (Jedis jedis = pool.getResource()) {
            jedis.auth("123456");
            String ping = jedis.ping();
            System.out.println(ping);
      }
}
```

JedisPool 也可以使用 `GenericObjectPoolConfig` 来初始化，支持更丰富的配置，如下：

```java
@Test
public void testJedisPool3() {
      GenericObjectPoolConfig config = new GenericObjectPoolConfig();
      // 连接池最大空闲数
      config.setMaxIdle(300);
      // 最大连接数
      config.setMaxTotal(1000);
      // 连接最大等待时间，如果是 -1 表示没有限制
      config.setMaxWaitMillis(30000);
      // 在空闲时检查有效性
      config.setTestOnBorrow(true);
      JedisPool pool = new JedisPool(config, "192.168.71.62", 6379, Protocol.DEFAULT_TIMEOUT, "123456");
      try (Jedis jedis = pool.getResource()) {
            String r = jedis.ping();
            System.out.println(r);
      }
}
```

### 2.3 封装

上面的代码无法实现强约束，下面我们可以做进一步的封装：

```java
public interface CallWithJedis {
    void call(Jedis jedis);
}
```

```java
public class Redis {
    private JedisPool pool;

    public Redis() {
        GenericObjectPoolConfig config = new GenericObjectPoolConfig();
        // 连接池最大空闲数
        config.setMaxIdle(300);
        // 最大连接数
        config.setMaxTotal(1000);
        // 连接最大等待时间，如果是 -1 表示没有限制
        config.setMaxWaitMillis(30000);
        // 在空闲时检查有效性
        config.setTestOnBorrow(true);
        /**
         * 1. Redis 地址
         * 2. Redis 端口
         * 3. 连接超时时间
         * 4. 密码
         */
        pool = new JedisPool(config, "192.168.71.62", 6379, 30000, "123456");
    }

    public void execute(CallWithJedis callWithJedis) {
        try (Jedis jedis = pool.getResource()) {
            callWithJedis.call(jedis);
        }
    }

    public static void main(String[] args) {
        Redis redis = new Redis();
        redis.execute(jedis -> {
            System.out.println(jedis.ping());
        });
    }
}
```

## 3 Lettuce

Lettuce 的 GitHub 地址：[https://github.com/lettuce-io/lettuce-core](https://github.com/lettuce-io/lettuce-core) 。

Lettuce 和 Jedis 的比较：

1. Jedis 在实现的过程中是直接连接 Redis 的，在多个线程之间共享一个 Jedis 实例，这是线程不安全的，如果想在多线程场景下使用 Jedis ，就得使用连接池，这样，每个线程都有自己的 Jedis 实例。
2. Lettuce 基于目前很火的 Netty NIO 框架来构建，所以克服了 Jedis 中线程不安全的问题， Lettuce 支持同步、异步以及响应式调用，多个线程可以共享一个连接实例。

首先创建一个普通的 Maven 项目 `redis-lettuce` ，项目创建成功后，添加 `Lettuce` 依赖： 

```xml
<dependencies>
      <dependency>
            <groupId>io.lettuce</groupId>
            <artifactId>lettuce-core</artifactId>
            <version>5.2.2.RELEASE</version>
      </dependency>
      <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13</version>
            <scope>test</scope>
      </dependency>
</dependencies>
```

然后创建一个测试类及测试方法：

```java
public class LettuceTests {
    @Test
    public void testLettuce() {
        RedisClient redisClient = RedisClient.create("redis://123456@192.168.71.62");
        StatefulRedisConnection<String, String> connect = redisClient.connect();
        RedisCommands<String, String> sync = connect.sync();
        sync.set("k1", "hellolettuce");
        String v = sync.get("k1");
        System.out.println(v);
    }
}
```

---

- [Redis 教程合集](https://mp.weixin.qq.com/s/iivXrj1cfTiPy89ueE_53Q)（微信左下方**阅读全文**可直达）。
- 本文示例代码：[https://github.com/cxy35/samples/tree/master/redis/redis-jedis](https://github.com/cxy35/samples/tree/master/redis/redis-jedis)
- 本文示例代码：[https://github.com/cxy35/samples/tree/master/redis/redis-lettuce](https://github.com/cxy35/samples/tree/master/redis/redis-lettuce)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)