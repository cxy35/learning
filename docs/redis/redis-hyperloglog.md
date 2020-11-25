---
title: Redis 实现 UV 统计
date: 2020-07-11 19:48:07
categories: Redis
tags: [Redis, HyperLogLog]
toc: true
---
本文学习在 Redis 中通过 `HyperLogLog` 实现 UV 统计。
<!-- more -->

## 1 概述

一般我们评估一个网站的访问量，有几个主要的参数：`PV（Page View）`网页的浏览量、`UV（User View）`访问的用户量。有很多第三方工具可以统计，如 cnzz，友盟等。如果自己实现的话，PV 比较简单，可以直接通过 Redis 计数器实现。但是 UV 就不一样，UV 涉及到去重的问题。

常规思路：我们首先需要在前端给每一个用户生成一个唯一 id，无论是登录用户还是未登录用户都需要，这个 id 伴随着请求一起到达后端，在后端我们可以通过 `Set` 数据结构中的 `sadd` 命令来存储这个 id，最后通过 `scard` 统计集合大小，进而得出 UV 数据。

按上述思路，如果是千万级别的 UV，需要的存储空间就非常惊人，用 Set 就不是很合适了。一般来说像 UV 统计这种，不需要特别精确，比如 800W 和 803W 的 UV，其实差别不大。因此，我们可以使用 `HyperLogLog` 来高效的实现。

## 2 基本使用

Redis 中提供的 HyperLogLog 就是专门用来解决上述问题的，HyperLogLog 提供了一套不怎么精确但是够用的去重方案，会有误差，官方给出的误差数据是 `0.81%`，这对于 UV 的统计够用了。

HyperLogLog 主要提供了以下命令：

- `pfadd`：用来添加记录，类似于 sadd ，添加过程中，重复的记录会自动去重。
- `pfcount`：则用来统计数据。
- `pfmerge`：合并多个统计结果，在合并的过程中，会自动去重多个集合中重复的元素。

数据量少的时候看不出来误差，我们在 Java 中多添加几个元素： 

```java
public class HyperLogLogTest {
    @Test
    public void testHyperLogLog() {
        Redis redis = new Redis();
        redis.execute(jedis -> {
            for (int i = 0; i < 1000; i++) {
                // 重复加入数据，理论值上总数为 1001
                jedis.pfadd("uv", "u" + i, "u" + (i + 1));
            }
            long uv = jedis.pfcount("uv");
            System.out.println(uv);
        });
    }
}
```

理论值上总数为 1001，实际打印出来 994，有误差，但是在可以接受的范围内。

---

- [Redis 教程合集](https://mp.weixin.qq.com/s/iivXrj1cfTiPy89ueE_53Q)（微信左下方**阅读全文**可直达）。
- 本文示例代码：[https://github.com/cxy35/samples/tree/master/redis/redis-jedis](https://github.com/cxy35/samples/tree/master/redis/redis-jedis)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)