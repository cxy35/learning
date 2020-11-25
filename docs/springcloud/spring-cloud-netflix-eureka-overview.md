---
title: Spring Cloud Netflix Eureka 概述
date: 2020-04-07 15:32:18
categories: Spring Cloud
tags: [Spring Cloud, Eureka]
toc: true
---
Eureka 是 Spring Cloud 中的服务注册中心，类似于 Dubbo 中的 Zookeeper 。本文学习 Eureka 概述、工作细节、集群等。它是 Netflix 家族成员之一。
<!-- more -->

## 1 服务注册中心

什么是注册中心，我们为什么需要注册中心？我们首先来看一个传统的单体应用：

![](https://oscimg.oschina.net/oscnet/up-5a6a0a6cd084e644ff2a29b7ab774c13aef.png)

在单体应用中，所有的业务都集中在一个项目中，当用户从浏览器发起请求时，直接由前端发起请求给后端，后端调用业务逻辑，给前端请求做出响应，完成一次调用。整个调用过程是一条直线，不需要服务之间的中转，所以没有必要引入注册中心。

随着公司项目越来越大，我们会将系统进行拆分，例如一个电商项目，可以拆分为订单模块、物流模块、支付模块、 CMS 模块等。这样，当用户发起请求时，就需要各个模块之间进行协作，这样不可避免的要进行模块之间的调用。此时，我们的系统架构就会发生变化：

![](https://oscimg.oschina.net/oscnet/up-88b13599539e855487f181729dcdf460c13.png)

在这里，大家可以看到，**模块之间的调用，变得越来越复杂，而且模块之间还存在强耦合**。例如 A 调用 B ，那么就要在 A 中写上 B 的地址，也意味着 B 的部署位置要固定，同时，如果以后 B 要进行集群化部署， A 也需要修改，非常麻烦，此时就需要注册中心了。

## 2 Eureka

### 2.1 Eureka 概述
 
Eureka 是 Netﬂix 公司提供的一款服务注册中心， **Eureka 基于 REST 来实现服务的注册与发现**，曾经的 Eureka 是 Spring Cloud 中最重要的核心组件之一。 Spring Cloud 中封装了 Eureka，在 Eureka 的基础上，优化了一些配置，然后提供了可视化的页面，可以方便的查看服务的注册情况以及服务注册中心集群的运行情况。

Eureka 由两部分：**服务端和客户端**，服务端就是注册中心，用来接收其他服务的注册，客户端则是一个 Java 客户端，需要向服务端注册，并可以实现负载均衡等功能。

![](https://oscimg.oschina.net/oscnet/up-c4d276e9a1b5ad5d78a17fd706330ba31d0.png)

从图中我们可以看出 Eureka 中有三个角色：

- `Eureka Server` ：注册中心
- `Eureka Provider` ：服务提供者
- `Eureka Consumer` ：服务消费者

### 2.2 Eureka 工作细节

Eureka 本身可以分为两大部分： `Eureka Server` 和 `Eureka Client` 。

#### 2.2.1 Eureka Server

Eureka Server 主要对外提供了三个功能：

- **服务注册**：所有的服务都注册到 Eureka Server 上面来。
- **提供注册表**：注册表就是所有注册上来服务的一个列表， Eureka Client 在调用服务时，需要获取这个注册表，一般来说，这个注册表会缓存下来，如果缓存失效，则直接获取最新的注册表。
- **同步状态**： Eureka Client 通过注册、心跳等机制，和 Eureka Server 同步当前客户端的状态。

#### 2.2.2 Eureka Client

Eureka Client 主要是用来简化每一个服务和 Eureka Server 之间的交互。 Eureka Client 会自动拉取、更新以及缓存 Eureka Server 中的信息，这样，即使 Eureka Server 所有节点都宕机， Eureka Client 依然能够获取到想要调用服务的地址（但是地址可能不准确）。

- **服务注册**

服务提供者将自己注册到服务注册中心（ Eureka Server ），需要注意，所谓的服务提供者，只是一个业务上的划分，本质上就是一个 Eureka Client 。当 Eureka Client 向 Eureka Server 注册时，他需要提供自身的一些元数据信息，例如 IP 地址、端口、名称、运行状态等。

- **服务续约**

Eureka Client 注册到 Eureka Server 上之后，事情没有结束，刚刚开始而已。注册成功后，默认情况下， Eureka CLient 每隔 30 秒就要向 Eureka Server 发送一条心跳消息，来告诉 Eureka Server 我还在运行。如果 Eureka Server 连续 90 秒都有没有收到 Eureka Client 的续约消息（连续三次没发送），它会认为 Eureka Client 已经掉线了，会将掉线的 Eureka Client 从当前的服务注册列表中剔除。
 
服务续约有两个相关的属性（一般不建议修改）：

```properties
# 服务续约时间，默认是 30 秒
eureka.instance.lease-renewal-interval-in-seconds=30
# 服务失效时间，默认是 90 秒
eureka.instance.lease-expiration-duration-in-seconds=90
```

- **服务下线**

当 Eureka Client 下线时，它会主动发送一条消息，告诉 Eureka Server ，我下线了。

- **获取注册表信息**

Eureka Client 从 Eureka Server 上获取服务的注册信息，并将其缓存在本地。本地客户端，在需要调用远程服务时，会从该信息中查找远程服务所对应的 IP 地址、端口等信息。 Eureka Client 上缓存的服务注册信息会定期更新( 30 秒)，如果 Eureka Server 返回的注册表信息与本地缓存的注册表信息不同的
话， Eureka Client 会自动处理。

这里也涉及到两个属性：

```properties
# 是否允许获取注册表信息
eureka.client.fetch-registry=true
# Eureka Client 上缓存的服务注册信息，定期更新的时间间隔，默认 30 秒
eureka.client.registry-fetch-interval-seconds=30
```

### 2.3 Eureka 集群

![](https://oscimg.oschina.net/oscnet/up-69602682fd18c88d4e724dcd6fcf9781025.png)

在这个集群架构中， Eureka Server 之间通过 Replicate 进行数据同步，**不同的 Eureka Server 之间不区分主从节点，所有节点都是平等的**。节点之间，通过置顶 serviceUrl 来互相注册，形成一个集群，进而提高节点的可用性。
 
在 Eureka Server 集群中，如果有某一个节点宕机， Eureka Client 会自动切换到新的 Eureka Server 上。每一个 Eureka Server 节点，都会互相同步数据。Eureka Server 的连接方式，可以是单线的，如： A-->B-->C ，此时， A 的数据也会和 C 之间互相同步。但是一般不建议这种写法，在我们配置 serviceUrl 时，可以指定多个注册地址，即 A 可以即注册到 B 上，也可以同时注册到 C 上。

Eureka 分区：

1. `region` ：地理上的不同区域。
2. `zone` ：具体的机房。

---

- [Spring Cloud 教程合集](https://mp.weixin.qq.com/s/SBmcs2bxumhNz4kky1pl-A)（微信左下方**阅读全文**可直达）。
- Spring Cloud 教程合集示例代码：[https://github.com/cxy35/spring-cloud-samples](https://github.com/cxy35/spring-cloud-samples)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)