---
title: Redis 实现消息队列
date: 2020-06-27 19:48:07
categories: Redis
tags: [Redis]
toc: true
---
本文学习在 Redis 中通过 `List/ZSet` 实现消息队列。
<!-- more -->

## 1 概述
 
我们平时使用的消息队列有 `RabbitMQ`、`RocketMQ`、`ActiveMQ` 以及大数据里边的 `Kafka`，他们都非常专业，提供了很多功能。如果我们的需求或场景非常简单，用他们就有点大材小用了，比如我们只需要 1 个消息队列，且只有 1 个消费者，类似这种简单情况我们可以直接使用 `Redis` 来做消息队列。

## 2 基本使用

### 2.1 消息队列

Redis 作为消息队列，我们可以使用 `List` 数据结构来实现，通过 `lpush/rpush` 命令来实现入列， `lpop/rpop` 命令来实现出列。

在 Java 客户端，我们一般会维护一个死循环来不停的从队列中读取消息，并处理，如果队列中有消息，则直接获取到，如果没有消息，就会陷入死循环，直到下一次有消息进入，这种死循环会造成大量的资源浪费，这个时候，我们可以使用之前讲的 `blpop/brpop` 。

### 2.2 延迟消息队列

Redis 作为延迟消息队列，我们可以使用 `ZSet` 数据结构来实现，因为 ZSet 中有一个 `score`，我们可以把时间作为 score，将 value 存到 redis 中，然后通过轮询的方式，去不断的读取消息出来。如果消息是一个字符串，直接发送即可，如果是一个对象，则需要对对象进行序列化，这里我们使用 JSON 来实现序列化和反序列化。

首先，在项目中，添加 JSON 依赖：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.10.3</version>
</dependency>
```

接着，构造一个消息对象：

```java
public class MyMessage {
    private String id;
    private Object data;

    @Override
    public String toString() {
        return "MyMessage{" +
                "id='" + id + '\'' +
                ", data=" + data +
                '}';
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public Object getData() {
        return data;
    }

    public void setData(Object data) {
        this.data = data;
    }
}
```

接着，封装一个延迟消息队列：

```java
public class MyDelayMq {
    private Jedis jedis;
    private String queue;

    public MyDelayMq(Jedis jedis, String queue) {
        this.jedis = jedis;
        this.queue = queue;
    }

    /**
     * 消息入列
     *
     * @param data 要发送的消息
     */
    public void enqueue(Object data) {
        // 构造一个 MyMessage
        MyMessage msg = new MyMessage();
        msg.setId(UUID.randomUUID().toString());
        msg.setData(data);
        // 序列化
        try {
            String s = new ObjectMapper().writeValueAsString(msg);
            System.out.println("send msg: " + new Date());
            // 消息发送，score 延迟 5 秒
            jedis.zadd(queue, System.currentTimeMillis() + 5000, s);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
    }

    /**
     * 消息出列
     */
    public void dequeue() {
        while (!Thread.interrupted()) {
            // 读取 score 在 0 到当前时间戳之间的消息
            Set<String> zrange = jedis.zrangeByScore(queue, 0, System.currentTimeMillis(), 0, 1);
            if (zrange.isEmpty()) {
                // 如果消息是空的，则休息 500 毫秒然后继续
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    break;
                }
                continue;
            }
            // 如果读取到了消息，则直接读取消息出来
            String s = zrange.iterator().next();
            if (jedis.zrem(queue, s) > 0) {
                // 抢到了，接下来处理业务
                try {
                    MyMessage msg = new ObjectMapper().readValue(s, MyMessage.class);
                    System.out.println("receive msg: " + new Date() + " : " + msg);
                } catch (JsonProcessingException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

最后，新增测试类：

```java
public class DelayMqTest {
    @Test
    public void testDelayMq() {
        Redis redis = new Redis();
        redis.execute(jedis -> {
            // 构造一个消息队列
            MyDelayMq queue = new MyDelayMq(jedis, "mq-delay");
            // 构造消息生产者
            Thread producer = new Thread() {
                @Override
                public void run() {
                    for (int i = 0; i < 5; i++) {
                        queue.enqueue("http://cxy35.com >>>> " + i);
                    }
                }
            };

            // 构造一个消息消费者
            Thread consumer = new Thread() {
                @Override
                public void run() {
                    queue.dequeue();
                }
            };

            // 启动
            producer.start();
            consumer.start();

            // 休息 7 秒后，停止程序
            try {
                Thread.sleep(7000);
                consumer.interrupt();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
    }
}
```

---

- [Redis 教程合集](https://mp.weixin.qq.com/s/iivXrj1cfTiPy89ueE_53Q)（微信左下方**阅读全文**可直达）。
- 本文示例代码：[https://github.com/cxy35/samples/tree/master/redis/redis-jedis](https://github.com/cxy35/samples/tree/master/redis/redis-jedis)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)