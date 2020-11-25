---
title: Spring Boot 整合 Redis + Spring Session（实现 Session 共享）
date: 2019-12-17 18:03:48
categories: Spring Boot
tags: [Spring Boot, Redis, Session]
toc: true
---
学习在 Spring Boot 中整合 Redis + Spring Session ，实现 Session 共享。先来回顾下在 SSM 中使用 Spring Session 的配置，首先是 web.xml 配置代理过滤器，然后在 Spring 容器中配置 Redis，最后再配置 Spring Session ，步骤有些繁琐。下面来看下在 Spring Boot 中如何使用，比较起来你会发现超级简单。
<!-- more -->

## 1 概述

在传统的单服务架构中，一般来说，只有一个服务器，那么不存在 Session 共享问题，但是在分布式/集群项目中， Session 共享则是一个必须面对的问题，先看一个简单的架构图：

![](https://oscimg.oschina.net/oscnet/up-d694ab58a043229967fabc06e897da41e80.png)

在这样的架构中，会出现一些单服务中不存在的问题，例如客户端发起一个请求，这个请求到达 Nginx 上之后，被 Nginx 转发到 Tomcat A 上，然后在 Tomcat A 上往 Session 中保存了一份数据，下次又来一个请求，这个请求被转发到 Tomcat B 上，此时再去 Session 中获取数据，发现没有之前的数据。对于这一类问题的解决，思路很简单，就是**将各个服务之间需要共享的数据，保存到一个公共的地方（主流方案就是 Redis ）**，如下：

![](https://oscimg.oschina.net/oscnet/up-27a07f3ef0fe6414a6170b49dd7fb8cf13e.png)

当所有 Tomcat 需要往 Session 中写数据时，都往 Redis 中写，当所有 Tomcat 需要读数据时，都从 Redis 中读。这样，不同的服务就可以使用相同的 Session 数据了。

这样的方案，可以由开发者手动实现，即手动往 Redis 中存储数据，手动从 Redis 中读取数据，相当于使用一些 Redis 客户端工具来实现这样的功能，毫无疑问，手动实现工作量还是蛮大的。

一个简化的方案就是使用 Spring Session 来实现这一功能， Spring Session 就是使用 Spring 中的代理过滤器，将所有的 Session 操作拦截下来，自动的将数据同步到 Redis 中，或者自动的从 Redis 中读取数据。

对于开发者来说，所有关于 Session 同步的操作都是透明的，开发者使用 Spring Session ，一旦配置完成后，具体的用法就像使用一个普通的 Session 一样。

## 2 创建工程并配置

创建 Spring Boot 项目 `spring-boot-redis-springsession` ，添加 `Web/Redis/Spring Session` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-3c699be8c83873c3e5d868aed6b3d42dcc8.png)

之后手动在 pom 文件中添加 `commos-pool2` 依赖，最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session-data-redis</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-pool2</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        <exclusions>
            <exclusion>
                <groupId>org.junit.vintage</groupId>
                <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```

接着在 `application.properties` 配置文件中添加 Redis 相关信息的配置和 Redis 连接池的配置，如下：

```properties
# Redis 配置
spring.redis.host=192.168.71.62
spring.redis.port=6379
spring.redis.database=0
spring.redis.password=000000

# 连接池配置， Spring Boot 默认用的是 lettuce ，而不是 Jedis ，需增加 commons-pool2 依赖
spring.redis.lettuce.pool.min-idle=5
spring.redis.lettuce.pool.max-idle=10
spring.redis.lettuce.pool.max-active=8
spring.redis.lettuce.pool.max-wait=1ms
spring.redis.lettuce.shutdown-timeout=100ms
```

## 3 使用

配置完成后 ，就可以使用 Spring Session 了，其实就是使用普通的 HttpSession ，其他的 Session 同步到 Redis 等操作，框架已经自动帮你完成了。新建 HelloController ，如下：

```java
@RestController
public class HelloController {

    // java -jar spring-boot-redis-springsession-0.0.1-SNAPSHOT.jar -- server.port=8080
    // java -jar spring-boot-redis-springsession-0.0.1-SNAPSHOT.jar -- server.port=8081
    @Value("${server.port}")
    Integer port;

    @GetMapping("/set")
    public String set(HttpSession session) {
        session.setAttribute("name", "cxy35");
        return String.valueOf(port);
    }

    @GetMapping("/get")
    public String get(HttpSession session) {
        return ((String) session.getAttribute("name")) + port;
    }
}
```

项目打包之后，分别在 `8080` 和 `8081` 端口启动服务。先访问 [http://127.0.0.1:8080/set](http://127.0.0.1:8080/set) 接口向 8080 这个服务的 Session 中保存一个变量，访问完成后，数据就已经自动同步到 Redis 中了。然后再访问 [http://127.0.0.1:8081/get](http://127.0.0.1:8081/get) 接口，发现可以获取到 8080 服务的 Session 中的数据，大功告成。

另外，上面是手动切换服务来实现测试，略显麻烦，可引入 `Nginx` 来实现服务实例自动切换。关于 Nginx 的使用可以参考 [Nginx 教程合集](https://mp.weixin.qq.com/s/TdLki2vnjW4hKUz_BgzEHg) 。

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-nosql/spring-boot-redis-springsession](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-nosql/spring-boot-redis-springsession)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)