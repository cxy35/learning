---
title: Spring Cloud Gateway 服务网关
date: 2020-05-04 18:54:08
categories: Spring Cloud
tags: [Spring Cloud, Gateway]
toc: true
---
学习在 Spring Cloud 中使用 Gateway 实现服务网关，包括基本使用、自动代理、 Predicate 、 Filter 等功能。
<!-- more -->

## 1 概述

Gateway 的主要功能如下：

- 限流
- 路径重写
- 动态路由
- 集成 Spring Cloud DiscoveryClient
- 集成 Hystrix 断路器

和 Zuul 相比，有如下区别：

1. Zuul 是 Netﬂix 公司的开源产品， Spring Cloud Gateway 是 Spring 家族中的产品，可以和 Spring 家族中的其他组件更好的融合。
2. Zuul 不支持长连接（版本一），例如 Websocket 。
3. Spring Cloud Gateway 支持限流。
4. Spring Cloud Gateway 基于 Netty 来开发，实现了异步和非阻塞，占用资源更小，性能强于 Zuul 。

## 2 准备工作

### 2.1 服务注册

创建 Spring Boot 项目 `gateway-client-provider` ，作为我们的**服务提供者**，添加 `Web/Eureka Client` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-0ccb137be7b2d3ed22cc9288f7a31927aa4.png)

最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
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

---

项目创建成功后，修改 `application.properties` 配置文件，将 gateway-client-provider 注册到 Eureka Server 上（服务注册中心使用 Eureka Server ），如下：

```properties
# 当前服务的名称
spring.application.name=gateway-client-provider
# 当前服务的端口
server.port=7000

# 服务注册中心地址
eureka.client.service-url.defaultZone=http://127.0.0.1:1111/eureka
```

接下来，启动 Eureka Server ，待服务注册中心启动成功后，再启动 gateway-client-provider ，两者都启动成功后，访问 [http://127.0.0.1:1111](http://127.0.0.1:1111) 可以看到 gateway-client-provider 的注册信息。

---

当然 gateway-client-provider 也可以集群化部署，下面对 gateway-client-provider 进行打包，之后我们在命令行启动两个 provider 实例：
 
```bash
java -jar gateway-client-provider-0.0.1-SNAPSHOT.jar --server.port=7000
java -jar gateway-client-provider-0.0.1-SNAPSHOT.jar --server.port=7001
```

---

最后在 gateway-client-provider 提供 hello 和 hello2 接口，用于后续服务消费者 gateway-client-consumer 来消费，如下：

```java
@RestController
public class ProviderController {
    @Value("${server.port}")
    Integer port; // 支持启动多个实例，做负载均衡，用端口区分

    @GetMapping("/hello")
    public String hello() {
        return "hello cxy35: " + port;
    }

    @GetMapping("/hello2")
    public String hello2(String name) {
        return "hello " + name;
    }
}
```

### 2.2 服务消费

创建 Spring Boot 项目 `gateway-client-consumer` ，作为我们的**服务消费者**，添加 `Eureka Client/Gateway` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-3b0bb9f59525519126319692ef6f5af1267.png)

最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
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

---

项目创建成功后，新建 `application.yml` 配置文件，将 gateway-client-consumer 注册到 Eureka Server 上（服务注册中心使用 Eureka Server ），如下：

```yml
# 当前服务的名称
spring:
  application:
    name: gateway-client-consumer
# 当前服务的端口
server:
  port: 7002

# 服务注册中心地址
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:1111/eureka
```

接下来，启动 gateway-client-consumer ，访问 [http://127.0.0.1:1111](http://127.0.0.1:1111) 可以看到 gateway-client-consumer 的注册信息。

## 3 基本使用

Spring Cloud Gateway 支持两种不同的配置方法：**编码配置、 YML 配置**。 

- 编码配置

在 `gateway-client-consumer` 项目启动类上配置一个 RouteLocator 类型的 Bean，就可以实现请求转发，如下：

```java
@SpringBootApplication
public class GatewayClientConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(GatewayClientConsumerApplication.class, args);
    }

    @Bean
    RouteLocator routeLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("cxy35_route", r ->
                        r.path("/get").uri("http://httpbin.org")) // 这是一个测试的地址
                .build();
    }
}
```

配置完成后，重启项目，访问 [http://127.0.0.1:7002/get](http://127.0.0.1:7002/get) 完成测试。

---

- YML 配置

注释掉上述 Java 代码，再修改 `gateway-client-consumer` 中的 `application.yml` 配置文件：

```yml
# 当前服务的名称
spring:
  application:
    name: gateway-client-consumer
  cloud:
    gateway:
      routes:
        - id: cxy35_route
          uri: http://httpbin.org # 这是一个测试的地址
          predicates:
            - Path=/get
# 当前服务的端口
server:
  port: 7002

# 服务注册中心地址
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:1111/eureka
```

配置完成后，重启项目，访问 [http://127.0.0.1:7002/get](http://127.0.0.1:7002/get) 完成测试。

## 4 结合微服务使用

修改 `gateway-client-consumer` 中的 `application.yml` 配置文件：

```yml
# 当前服务的名称
spring:
  application:
    name: gateway-client-consumer
  cloud:
    gateway:
      routes:
        - id: cxy35_route
          uri: http://httpbin.org # 这是一个测试的地址
          predicates:
            - Path=/get
      discovery:
        locator:
          enabled: true # 开启自动代理
# 当前服务的端口
server:
  port: 7002

# 服务注册中心地址
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:1111/eureka

logging:
  level:
    org.springframework.cloud.gateway: debug
```

接下来，就可以通过 Gateway 访问到其他注册在 Eureka 上的服务了，如 [http://127.0.0.1:7002/GATEWAY-CLIENT-PROVIDER/hello](http://127.0.0.1:7002/GATEWAY-CLIENT-PROVIDER/hello) ，注意大小写问题。

## 5 Predicate

路由配置 `Route` 中的 `Predicate` 支持多种配置方式：

1. 通过时间匹配： `After` （表示在某个时间点之前进行请求转发）/ `Before` （表示在某个时间点之前进行请求转发）/ `Between` （表示在两个时间点之间，两个时间点用 , 隔开）。

```yml
spring:
  cloud:
    gateway:
      routes:
        - id: cxy35_route2
          uri: http://httpbin.org
          predicates:
            - After=2021-01-01T01:01:01+08:00[Asia/Shanghai]
```

2. 通过请求方法匹配： `Method` 。

```yml
spring:
  cloud:
    gateway:
      routes:
        - id: cxy35_route2
          uri: http://httpbin.org
          predicates:
            - Method=GET
```

3. 通过请求路径匹配： `Path` 。

```yml
spring:
  cloud:
    gateway:
      routes:
        - id: cxy35_route2
          uri: http://httpbin.org
          predicates:
            - Path=/2020/01/{segment}
```

上述配置表示路径满足 `/2020/01/` 这个规则，都会被进行转发。

4. 通过请求参数名或值匹配： `Query` 。

```yml
spring:
  cloud:
    gateway:
      routes:
        - id: cxy35_route2
          uri: http://httpbin.org
          predicates:
            - Query=name
```

上述配置表示请求中一定要有 name 参数才会进行转发，否则不会进行转发。

也可以同时指定参数名和参数值，例如参数的 key 为 name ， value 必须要以 java 开始：

```yml
spring:
  cloud:
    gateway:
      routes:
        - id: cxy35_route2
          uri: http://httpbin.org
          predicates:
            - Query=name,java.*
```

5. 多种匹配方式组合使用。

```yml
spring:
  cloud:
    gateway:
      routes:
        - id: cxy35_route2
          uri: http://httpbin.org
          predicates:
            - After=2021-01-01T01:01:01+08:00[Asia/Shanghai]
            - Method=GET
            - Query=name,java.*
 ```

## 6 Filter

Spring Cloud Gateway 中的过滤器分为两大类： `GatewayFilter` / `GlobalFilter` 。

比如 `AddRequestParameter` 过滤器会在请求转发路由的时候，自动额外添加参数，如：

```yml
spring:
  cloud:
    gateway:
      routes:
        - id: cxy35_route3
          uri: lb://gateway-client-provider
          filters:
            - AddRequestParameter=name,cxy35
          predicates:
            - Method=GET
```

配置完成后，重启项目，访问 [http://127.0.0.1:7002/hello2](http://127.0.0.1:7002/hello2) 完成测试。

---

- [Spring Cloud 教程合集](https://mp.weixin.qq.com/s/SBmcs2bxumhNz4kky1pl-A)（微信左下方**阅读全文**可直达）。
- Spring Cloud 教程合集示例代码：[https://github.com/cxy35/spring-cloud-samples](https://github.com/cxy35/spring-cloud-samples)
- 本文示例代码：[https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-gateway](https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-gateway)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)