---
title: Spring Cloud Stream 构建消息驱动的微服务
date: 2020-05-16 18:15:01
categories: Spring Cloud
tags: [Spring Cloud, Stream]
toc: true
---
学习在 Spring Cloud 中使用 Stream 构建消息驱动的微服务，包括基本使用、自定义消息通道、消息分组、消息分区、定时任务等功能。
<!-- more -->

## 1 概述

Spring Cloud Stream 提供了一个微服务和消息中间件之间的粘合剂，这个粘合剂叫做 `Binder` ， Binder 负责与消息中间件进行交互。而开发者则通过 `inputs` 或者 `outputs` 这样的消息通道与 Binder 进行交互。

## 2 基本使用

创建 Spring Boot 项目 `spring-cloud-stream` ，添加 `Web/RabbitMQ/Cloud Stream` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-94c27791e3c3f4d1c95395c4b7e9ea0f71e.png)

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

项目创建成功后，在 `application.properties` 配置文件中配置 RabbitMQ 的基本配置信息：

```properties
spring.rabbitmq.host=127.0.0.1
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```

---

接下来，创建一个简单的消息接收器：

```java
@EnableBinding(Sink.class) // @EnableBinding 表示绑定 Sink 这个默认的消息通道
public class MsgReceiver {
    public final static Logger logger = LoggerFactory.getLogger(MsgReceiver.class);

    @StreamListener(Sink.INPUT)
    public void receive(Object payload) {
        logger.info("MsgReceiver: " + payload + " >>> " + new Date());
    }
}
```

启动 RabbitMQ 和项目，再访问 [http://127.0.0.1:15672](http://127.0.0.1:15672) ，在 RabbitMQ 后台管理页面发送一条消息，上述消息接收器中可以正常收到消息。

![](https://oscimg.oschina.net/oscnet/up-5b93453aa5477aad14dffc52c9603d3c961.png)

![](https://oscimg.oschina.net/oscnet/up-b0494ea2b629dede72036f1ebd894d8cdfa.png)

## 3 自定义消息通道

首先，创建一个名为 `MyChannel` 的接口，作为我们的消息通道：

```java
// 自定义消息通道
public interface MyChannel {
    String INPUT = "cxy35-input";
    String OUTPUT = "cxy35-output";

    @Output(OUTPUT)
    MessageChannel output();

    @Input(INPUT)
    SubscribableChannel input();
}
```

注意：

- 两个消息通道的名字是不一样的。
- 从 F 版开始，默认使用通道的名称作为实例命令，所以这里的通道名字不可以相同（早期版本可以相同），这样的话，为了能够正常收发消息，需要我们在 `application.properties` 中做一些额外配置。

```properties
# 绑定消息通道
spring.cloud.stream.bindings.cxy35-input.destination=cxy35-topic
spring.cloud.stream.bindings.cxy35-output.destination=cxy35-topic
```

---

接下来，自定义一个消息接收器，用来接收自己的消息通道里的消息：

```java
@EnableBinding(MyChannel.class) // @EnableBinding 表示绑定 MyMsgReceiver 这个自定义的消息通道
public class MyMsgReceiver {
    public final static Logger logger = LoggerFactory.getLogger(MyMsgReceiver.class);

    @StreamListener(MyChannel.INPUT)
    public void receive(Object payload) {
        logger.info("MyMsgReceiver: " + payload + " >>> " + new Date());
    }
}
```

再定义一个 HelloController 进行测试：

```java
@RestController
public class HelloController {
    @Autowired
    MyChannel myChannel;

    @GetMapping("/hello")
    public void hello() {
        myChannel.output().send(MessageBuilder.withPayload("Hello Stream").build());
    }
}
```

重启项目，访问 [http://127.0.0.1:8080/hello](http://127.0.0.1:8080/hello) ，上述自定义的消息接收器中可以正常收到消息。

## 4 消息分组

默认情况下，如果消费者是一个集群，一条消息会被多次消费。通过消息分组，我们可以解决这个问题。只需要添加如下配置即可：

```properties
# 设置消息分组
spring.cloud.stream.bindings.cxy35-input.group=g1
spring.cloud.stream.bindings.cxy35-output.group=g1
```

接下来，项目打包，启动两个实例。

```bash
java -jar spring-cloud-stream-0.0.1-SNAPSHOT.jar --server.port=8080
java -jar spring-cloud-stream-0.0.1-SNAPSHOT.jar --server.port=8081
```

重启项目，访问 [http://127.0.0.1:8080/hello](http://127.0.0.1:8080/hello) ，发现一条消息只会被其中某一个实例消费。

## 5 消息分区

通过消息分区可以实现相同特征的消息总是被同一个消费者（实例）处理。只需要添加如下配置即可：

```properties
# 设置消息分区
# 配置消费者：开启消息分区
spring.cloud.stream.bindings.cxy35-input.consumer.partitioned=true
# 配置消费者：消费者实例个数
spring.cloud.stream.instance-count=2
# 配置消费者：当前实例的下标，这里指定了一个，多实例启动时指定另一个
spring.cloud.stream.instance-index=0

# 配置生产者：消息被下标为 1 的消费者（实例）消费
spring.cloud.stream.bindings.cxy35-output.producer.partition-key-expression=1
# 配置生产者：消费端的节点数量
spring.cloud.stream.bindings.cxy35-output.producer.partition-count=2
```

接下来，项目打包，启动两个实例，启动时要动态修改 `spring.cloud.stream.instance-index` 参数的值。

```bash
java -jar spring-cloud-stream-0.0.1-SNAPSHOT.jar --server.port=8080 --spring.cloud.stream.instance-index=0
java -jar spring-cloud-stream-0.0.1-SNAPSHOT.jar --server.port=8081 --spring.cloud.stream.instance-index=1
```

重启项目，访问 [http://127.0.0.1:8080/hello](http://127.0.0.1:8080/hello) ，发现的消息只会被 8081 那个实例消费。

## 6 定时任务

每天定时执行的任务，可以使用 cron 表达式，但有一种比较特殊的定时任务，例如几分钟后执行，这种可以使用 `Spring Cloud Stream + RabbitMQ` 来实现。先下载 RabbitMQ 相关插件：[https://dl.bintray.com/rabbitmq/community-plugins/3.7.x/rabbitmq_delayed_message_exchange/rabbitmq_delayed_message_exchange-20171201-3.7.x.zip](https://dl.bintray.com/rabbitmq/community-plugins/3.7.x/rabbitmq_delayed_message_exchange/rabbitmq_delayed_message_exchange-20171201-3.7.x.zip) 。

执行如下命令：

```bash
# 解压下载的文件
unzip rabbitmq_delayed_message_exchange-20171201-3.7.x.zip
# 将解压后的文件，拷贝到 Docker 容器中
docker cp /root/rabbitmq_delayed_message_exchange-20171201-3.7.x.ez cxy35-rabbit:/plugins
# 进入到容器中
docker exec -it cxy35-rabbit /bin/bash
# 启用插件
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
# 查看是否启用成功
rabbitmq-plugins list
```

---

接着，修改配置文件，开启消息延迟功能：

```properties
# 开启消息延迟功能
spring.cloud.stream.rabbit.bindings.cxy35-input.consumer.delayed-exchange=true
spring.cloud.stream.rabbit.bindings.cxy35-output.producer.delayed-exchange=true
```

修改消息输入输出通道的 destination 定义：

```properties
spring.cloud.stream.bindings.cxy35-input.destination=delay_msg
spring.cloud.stream.bindings.cxy35-output.destination=delay_msg
```

然后，增加 hello2 接口，在消息发送时，设置消息延迟时间为 3 秒：

```java
@RestController
public class HelloController {
    public final static Logger logger = LoggerFactory.getLogger(HelloController.class);

    @Autowired
    MyChannel myChannel;

    @GetMapping("/hello")
    public void hello() {
        myChannel.output().send(MessageBuilder.withPayload("Hello Stream").build());
    }

    @GetMapping("/hello2")
    public void hello2() {
        logger.info("send msg:" + new Date());
        myChannel.output().send(MessageBuilder.withPayload("Hello Stream 2").setHeader("x-delay", 3000).build());
    }
}
```

重启项目，访问 [http://127.0.0.1:8080/hello2](http://127.0.0.1:8080/hello2) ，发现消息在被发送 3 秒后才会被接收消费。

---

- [Spring Cloud 教程合集](https://mp.weixin.qq.com/s/SBmcs2bxumhNz4kky1pl-A)（微信左下方**阅读全文**可直达）。
- Spring Cloud 教程合集示例代码：[https://github.com/cxy35/spring-cloud-samples](https://github.com/cxy35/spring-cloud-samples)
- 本文示例代码：[https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-stream](https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-stream)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)