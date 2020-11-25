---
title: Spring Cloud Sleuth 链路追踪
date: 2020-05-17 18:27:29
categories: Spring Cloud
tags: [Spring Cloud, Sleuth]
toc: true
---
学习在 Spring Cloud 中使用 Sleuth 实现链路追踪，包括基本使用、异步任务、定时任务等功能，并结合 Zipkin 展示收集到的信息。
<!-- more -->

## 1 概述

在大规模的分布式系统中，一个完整的系统是由很多种不同的服务来共同支撑的。不同的系统可能分布在上千台服务器上，横跨多个数据中心。一旦系统出问题，此时问题的定位就比较麻烦。在微服务环境下，一次客户端请求，可能会引起数十次、上百次服务端服务之间的调用。一旦请求出问题了，我们需要考虑很多东西：

- 如何快速定位问题？
- 如果快速确定此次客户端调用，都涉及到哪些服务？
- 到底是哪一个服务出问题了？

要解决这些问题，就涉及到**分布式链路追踪**。分布式链路追踪系统主要用来跟踪服务调用记录的，一般来说，一个分布式链路追踪系统，有三个部
分功能：

- 数据收集。
- 数据存储。
- 数据展示。

`Spring Cloud Sleuth` 是 Spring Cloud 提供的一套分布式链路追踪系统，有 3 个核心概念：

- `Trace`：从请求到达系统开始，到给请求做出响应，这样一个过程成为 Trace 。
- `Span`：每次调用服务时，埋入的一个调用记录，成为 Span 。
- `Annotation`：相当于 Span 的语法，描述 Span 所处的状态。

## 2 基本使用

创建 Spring Boot 项目 `sleuth` ，添加 `Web/Sleuth` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-acc3d06472eeff38a4a4ae14119dc561b00.png)

最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-sleuth</artifactId>
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

项目创建成功后，在 `application.properties` 配置文件中配置服务名称，这个名称在输出的日志中会体现出来：

```properties
spring.application.name=sleuth
```

---

接下来创建一个 `HelloController` ，定义 hello 接口，打印日志测试（日志与 Sleuth 会自动整合）：

```java
@RestController
public class HelloController {
    private static final Logger logger = LoggerFactory.getLogger(HelloController.class);

    @GetMapping("/hello")
    public String hello() {
        logger.info("Hello Spring Cloud Sleuth");
        return "Hello Spring Cloud Sleuth";
    }
}
```

启动应用，访问 [http://127.0.0.1:8080/hello](http://127.0.0.1:8080/hello) 接口，结果如下：

![](https://oscimg.oschina.net/oscnet/up-dea20709cb038805203ffaa71e28c86bbb6.png)

可以看到 Spring Cloud Sleuth 的输出，其中 `[sleuth,9309020b37b78ad7,9309020b37b78ad7,false]` 表示：`服务名称,Trace Id,Span Id,是否暴露`。

---

然后，在启动类中定义 RestTemplate ，用于服务之间调用：

```java
@SpringBootApplication
public class SpringCloudSleuthApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudSleuthApplication.class, args);
    }

    @Bean
    RestTemplate restTemplate() {
        return new RestTemplate();
    }

}
```

再定义两个接口，在 hello2 中调用 hello3，形成调用链：

```java
@GetMapping("/hello2")
public String hello2() throws InterruptedException {
    logger.info("hello2");
    Thread.sleep(500);
    return restTemplate.getForObject("http://127.0.0.1:8080/hello3", String.class);
}

@GetMapping("/hello3")
public String hello3() throws InterruptedException {
    logger.info("hello3");
    Thread.sleep(500);
    return "hello3";
}
```

重新启动应用，访问 [http://127.0.0.1:8080/hello2](http://127.0.0.1:8080/hello2) 接口，结果如下：

![](https://oscimg.oschina.net/oscnet/up-a4dd2b02c2272f291b043d8a117ea21c5e7.png)

可以看到一个 Trace 由多个 Span 组成，一个 Trace 相当于就是一个调用链，而一个 Span 则是这个链中的每一次调用过程。

## 3 异步任务

Spring Cloud Sleuth 中也可以收集到异步任务中的信息。

首先，在启动类上通过 `@EnableAsync` 注解开启异步任务：

```java
@SpringBootApplication
@EnableAsync // 开启异步任务
public class SpringCloudSleuthApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudSleuthApplication.class, args);
    }

    @Bean
    RestTemplate restTemplate() {
        return new RestTemplate();
    }

}
```

接着，创建一个 HelloService ，提供一个异步任务方法：

```java
@Service
public class HelloService {
    private static final Logger logger = LoggerFactory.getLogger(HelloService.class);

    @Async
    public String backgroundFun() {
        logger.info("backgroundFun");
        return "backgroundFun";
    }
}
```

然后，在 HelloController 中调用该异步方法：

```java
@GetMapping("/hello4")
public String hello4() {
    logger.info("hello4");
    helloService.backgroundFun();
    return "hello4";
}
```

重新启动应用，访问 [http://127.0.0.1:8080/hello4](http://127.0.0.1:8080/hello4) 接口，结果如下：

![](https://oscimg.oschina.net/oscnet/up-abd95dac2593d9d2f4301c8df03b2c5968f.png)

可以看到异步任务会有一个单独的 Span Id 。

## 4 定时任务

Spring Cloud Sleuth 也可以收集定时任务的信息。

首先，在启动类上通过 `@EnableScheduling` 注解开启异步任务：

```java
@SpringBootApplication
@EnableAsync // 开启异步任务
@EnableScheduling // 开启定时任务
public class SpringCloudSleuthApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudSleuthApplication.class, args);
    }

    @Bean
    RestTemplate restTemplate() {
        return new RestTemplate();
    }

}
```

然后在 HelloSerivce 中，添加定时任务，去调用 background 方法。

```java
@Scheduled(cron = "0/10 * * * * ?")
public void sche1() {
    logger.info("start:");
    backgroundFun();
    logger.info("end:");
}
```

重新启动应用，结果如下：

![](https://oscimg.oschina.net/oscnet/up-fa70ff6fc235a7a540deaae6c3ac6ff90d8.png)

可以看到在定时任务中，每一次定时任务都会产生一个新的 Trace ，并且在调用过程中， Span Id 都是一致的，这个和普通的调用不一样。

## 5 Zipkin

Zipkin 本身是一个由 Twitter 公司开源的分布式追踪系统，分为 Server 端和 Client 端， Server 用来展示数据， Client 用来收集+上报数据。

### 5.1 准备工作

Zipkin 要先把数据存储起来，这里我们使用 Elasticsearch 来存储，首先安装 es 和 es-head（可视化工具，也可以使用 Kibana ）。

- es 安装命令：`docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.1.0`

- es-head 安装有三种方式：

    1. 直接下载软件安装。
    2. 通过 Docker 安装
    3. 安装 Chrome/Firefox 插件。

- RabbitMQ 安装：略。

- Zipkin 安装：`docker run -d -p 9411:9411 --name zipkin -e ES_HOSTS=127.0.0.1 -e STORAGE_TYPE=elasticsearch -e ES_HTTP_LOGGING=BASIC -e RABBIT_URI=amqp://guest:guest@127.0.0.1:5672 openzipkin/zipkin`

Zipkin 安装的参数说明：

- ES_HOSTS：es 的地址。
- STORAGE_TYPE：数据存储方式。
- RABBIT_URI：要连接的 Rabbit 的地址。

### 5.2 实践

创建 Spring Boot 项目 `sleuth-zipkin` ，添加 `Web/Sleuth/Zipkin Client/RabbitMQ/Cloud Stream` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-dbb510b934f70f483d10de8c8d3ad613407.png)

最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-sleuth</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-zipkin</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-stream</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
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
    <dependency>
        <groupId>org.springframework.amqp</groupId>
        <artifactId>spring-rabbit-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-stream-test-support</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

项目创建成功后，在 `application.properties` 配置文件中配置 `Sleuth/Zipkin Client/RabbitMQ` 等相关信息：

```properties
spring.application.name=sleuth-zipkin

# 开启链路追踪
spring.sleuth.web.client.enabled=true
# 配置采样比例，默认为 0.1
spring.sleuth.sampler.probability=1

# 开启 zipkin
spring.zipkin.enabled=true
# zipkin 地址
spring.zipkin.base-url=http://127.0.0.1:9411
# 追踪消息的发送类型
spring.zipkin.sender.type=rabbit

# 配置 RabbitMQ
spring.rabbitmq.host=127.0.0.1
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```
---

接下来提供一个测试的 HelloController：

```java
@RestController
public class HelloController {
    private static final Logger logger = LoggerFactory.getLogger(HelloController.class);

    @GetMapping("/hello")
    public String hello(String name) {
        logger.info("sleuth-zipkin-hello");
        return "hello " + name + " !";
    }
}
```

---

同理，再创建一个类似的 Spring Boot 项目 `sleuth-zipkin2` ，设置不同端口 `server.port=8081` ，并提供一个 /hello 接口通过 RestTemplate 调用 `sleuth-zipkin` 中的 /hello 接口，如下：

```java
@RestController
public class HelloController {
    private static final Logger logger = LoggerFactory.getLogger(HelloController.class);
    @Autowired
    RestTemplate restTemplate;

    @GetMapping("/hello")
    public void hello(String name) {
        String s = restTemplate.getForObject("http://127.0.0.1:8080/hello?name={1}", String.class, "cxy35");
        logger.info(s);
    }
}
```

启动 `sleuth-zipkin` 和 `sleuth-zipkin2` ，访问 [http://127.0.0.1:8081/hello](http://127.0.0.1:8081/hello) 之后，可以查看控制台 Sleuth 的输出信息、 Zipkin Server 端的信息 ([http://127.0.0.1:9411/zipkin](http://127.0.0.1:9411/zipkin))、 es-head (软件或插件) 或 Kibana ([http://127.0.0.1:5601/](http://127.0.0.1:5601/)) 中的信息、 RabbitMQ 中的信息 ([http://127.0.0.1:15672/](http://127.0.0.1:15672/))。

---

- [Spring Cloud 教程合集](https://mp.weixin.qq.com/s/SBmcs2bxumhNz4kky1pl-A)（微信左下方**阅读全文**可直达）。
- Spring Cloud 教程合集示例代码：[https://github.com/cxy35/spring-cloud-samples](https://github.com/cxy35/spring-cloud-samples)
- 本文示例代码：[https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-sleuth](https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-sleuth)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)