---
title: Spring Cloud Bus 消息总线
date: 2020-05-14 19:13:48
categories: Spring Cloud
tags: [Spring Cloud, Bus]
toc: true
---
学习在 Spring Cloud 中使用 Bus 实现消息总线，包括配置文件自动批量刷新、逐个刷新等功能。
<!-- more -->

## 1 概述

Spring Cloud Bus 通过轻量级的消息代理连接各个微服务，可以用来广播配置文件的更改，或者管理服务监控。在 [Spring Cloud Config 分布式配置中心](https://mp.weixin.qq.com/s/QRO0WBoPS_13IdK_VoAWzA) 一文中，我们提到，**当配置文件发生变化之后， conﬁg-server 可以及时感知到变化，但是 conﬁg-client 不会及时感知到变化**，默认情况下， conﬁg-client 只有重启才能加载到最新的配置文件。当时我们在 conﬁg-client 中结合 actuator 中的 refresh 来解决了这个问题，但是，如果 conﬁg-client 数量很多的时候，这种方案就显得很繁琐了，不合适。本文我们结合 Spring Cloud Bus 来解决这一问题。

## 2 基本使用

将 spring-cloud-config 中的代码（包括 conﬁg-server/conﬁg-client ）拷贝一份到 spring-cloud-bus 中（重命名为 bus-conﬁg-server/bus-conﬁg-client ），在此基础上进行修改。

首先，安装并启动 RabbitMQ 。

接着，在 bus-conﬁg-server 和 bus-conﬁg-client 中分别添加 `Spring Cloud Bus` 依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

然后，在 bus-conﬁg-server 和 bus-conﬁg-client 中分别添加 RabbitMQ 配置：

```properties
# 配置 RabbitMQ
spring.rabbitmq.host=127.0.0.1
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```

---

同时，给 bus-conﬁg-server 添加 actuator 依赖，将提供刷新接口：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

然后，修改 bus-conﬁg-server 中的 bootstrap.properties 配置文件，使 bus-refresh 端点暴露出来：

```properties
# 暴露 bus-refresh 端点
management.endpoints.web.exposure.include=bus-refresh
```

---

由于给 bus-conﬁg-server 中的所有接口都添加了保护，所以刷新接口将无法直接访问，此时，可以通过修改 Security 配置，对端点的权限做出修改：

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated()
                .and()
                .httpBasic()
                .and()
                .csrf().disable();
    }
}
```

在这段配置中，开启了 HttpBasic 登录，这样，在发送刷新请求时，就可以直接通过 HttpBasic 配置认证信息了。

最后分别启动 Eureka Server/bus-config-server/bus-config-client ，然后修改配置信息提交到 GitHub，访问 [http://127.0.0.1:8082/hello](http://127.0.0.1:8082/hello) 不会有变化。然后，调用 [http://127.0.0.1:8080/actuator/bus-refresh](http://127.0.0.1:8080/actuator/bus-refresh) [POST]。再次访问 [http://127.0.0.1:8082/hello](http://127.0.0.1:8082/hello) 发现有变化。

![](https://oscimg.oschina.net/oscnet/up-36b329d7776dd00530af075ab743f2fcf9f.png)

这个 POST 是针对 bus-conﬁg-server 的， bus-conﬁg-server 会把这个刷新的指令传到 RabbitMQ ，然后 RabbitMQ 再把指令传给各个 bus-conﬁg-client ，实现了配置文件自动批量刷新。

---

如果更新配置文件之后，不希望每一个微服务 bus-conﬁg-client 都去刷新配置文件，那么可以通过如下配置解决问题。

首先，修改 bus-conﬁg-client 中的 bootstrap.properties 配置文件，给每一个 bus-conﬁg-client 添加一个 instance-id：

```properties
eureka.instance.instance-id=${spring.application.name}:${server.port}
```

然后，对 bus-conﬁg-client 进行打包，打包完成后，通过如下命令启动两个 bus-conﬁg-client 实例：

```
java -jar bus-config-client-0.0.1-SNAPSHOT.jar --server.port=8082
java -jar bus-config-client-0.0.1-SNAPSHOT.jar --server.port=8083
```

修改配置文件，并且提交到 GitHub 之后，可以通过如下方式只刷新某一个微服务，例如只刷新 8082 的服务。

[http://127.0.0.1:8080/actuator/bus-refresh/bus-conﬁg-client:8082](http://127.0.0.1:8080/actuator/bus-refresh/bus-conﬁg-client:8082) [POST]

其中 bus-conﬁg-client:8082 表示服务的 instance-id 。

![](https://oscimg.oschina.net/oscnet/up-6c0e3dde51c83a412c15a5396868582ae31.png)

---

- [Spring Cloud 教程合集](https://mp.weixin.qq.com/s/SBmcs2bxumhNz4kky1pl-A)（微信左下方**阅读全文**可直达）。
- Spring Cloud 教程合集示例代码：[https://github.com/cxy35/spring-cloud-samples](https://github.com/cxy35/spring-cloud-samples)
- 本文示例代码：[https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-bus](https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-bus)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)