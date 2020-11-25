---
title: Spring Cloud 概述
date: 2020-03-20 15:27:23
categories: Spring Cloud
tags: [Spring Cloud]
toc: true
---
通过本文学习**微服务介绍**、 **Spring Cloud 介绍**，让大家对 Spring Cloud 有个初步的认识。
<!-- more -->

## 1 微服务介绍

2009 年， `Netflix` 公司重新定义了它的应用程序员的开发模型，这个算是微服务的首次探索。

2014 年， [《Microservices》](https://www.martinfowler.com/articles/microservices.html) 这篇文章以一个更加通俗易懂的方式，为大家定义了微服务。

**互联网应用**产品的两大特点：

- 需求变化快。
- 用户群体庞大。

在这样的情况下，我们需要构建一个能够**灵活扩展**，同时能够**快速应对外部环境变化**的一个应用，使用传统的开发方式，显然无法满足需求。这个时候，微服务就登场了。

### 1.1 什么是微服务

简单来说，微服务就是一种**将一个单一应用程序拆分为一组小型服务**的方法，拆分完成后，每一个服务都运行在独立的进程中，**服务与服务之间采用轻量级的通信机制来进行沟通**（其中 Spring Cloud 中采用基于 HTTP 的 RESTful API ）。

每一个服务，都是围绕具体的业务进行构建，例如一个电商系统，包括：订单服务、支付服务、物流服务、会员服务等，这些拆分后的应用都是独立的应用，都可以独立的部署到生产环境中。同时在采用微服务之后，我们的项目不再拘泥于一种语言，可以是 Java/Go/Python/PHP 等混合使用，这在传统的应用开发中，是无法想象的。而使用了微服务之后，我们可以根据业务上下文来选择合适的语言和构建工具进行构建。

微服务可以理解为是 `SOA` 的一个传承，一个本质的区别是微服务是一个真正分布式、去中心化的，微服务的拆分比 SOA 更加彻底。

### 1.2 微服务的优势

- 复杂度可控。
- 独立部署。
- 技术选型灵活。
- 较好的容错性。
- 较强的可扩展性。

### 1.3 微服务的特性

- 不主动
- 不拒绝
- 不负责

## 2 Spring Cloud 介绍

Spring Cloud 可以理解为微服务这种思想在 Java 领域的一个具体落地。 Spring Cloud 在发展之初，就借鉴了微服务的思想，同时结合 Spring Boot 让 Spring Cloud 具备了组件的一键式启动和部署的能力，极大的简化了微服务架构的落地。

Spring Cloud 这种框架，从设计之初，就充分考虑了分布式架构演化所需要的功能，例如服务注册、配置中心、消息总线、负载均衡等。这些功能都是以可插拔的形式提供出来的，这样，在分布式系统不断演化的过程中，我们的 Spring Cloud 也可以非常方便的进化。

### 2.1 什么是 Spring Cloud

Spring Cloud 是一系列框架的集合， 内部包含了许多框架，这些框架互相协作，共同来构建**分布式系统**。利用这些组件，可以非常方便的构建一个分布式系统。

### 2.2 核心特性

- 分布式配置中心。
- 服务注册与发现。
- 链路器。
- 服务调用。
- 负载均衡。
- 断路器/熔断机制。
- 全局锁。
- 选举与集群状态管理。
- 分布式消息/消息总线。

### 2.3 版本名称

不同于其他的框架， Spring Cloud 的版本名称是通过 A(Angel)、 B(Brixton)、 C(Camden)、D(Dalston)、 E(Edgware)、 F(Finchley) 这样来命名的，这些名字使用了伦敦地铁站的名
字，目前最新版是 `Hoxton SR3` 。

Spring Cloud 中，除了大的版本之外，还有一些小版本，小版本命名方式如下：

- `M`: 是 Milestone 的缩写，如: M1/M2。
- `RC`: 是 Release Candidate 的缩写，表示该项目处于候选状态，这是正式发版之前的一个状态，如: RC1/RC2 。
- `SR`: 是 Service Release 的缩写，表示项目正式发布的稳定版，相当于 GA(Generally Available) 版。如: SR1/SR2 。
- `SNAPSHOT`: 表示快照版。

### 2.4 组件

- `Spring Cloud Netflix`: 这个组件，在 Spring Cloud 成立之初，立下了汗马功劳。但 2018 年的断更事件，使得 Netflix 走下神坛。
- `Spring Cloud Config`: 分布式配置中心，利用 Git/SVN 来集中管理项目的配置文件。
- `Spring Cloud Bus`: 消息总线，可以构建消息驱动的微服务，也可以用来做一些状态管理等。
- `Spring Cloud Consul`: 服务注册与发现。
- `Spring Cloud Stream`: 基于 Redis/RabbitMQ/Kafka 实现的消息微服务。
- `Spring Cloud OpenFeign`: 提供 OpenFeign 集成到 Spring Boot 应用中的方式，主要解决微服务之间的调用问题。
- `Spring Cloud Gateway`: Spring Cloud 官方推出的网关服务。
- `Spring Cloud Cloudfoundry`: 利用 Cloudfoundry 集成我们的应用程序。
- `Spring Cloud Security`: 在 Zuul 代理中，为 OAuth2 客户端认证提供支持。
- `Spring Cloud AWS`: 快速集成亚马逊云服务。
- `Spring Cloud Contract`: 一个消费者驱动的、面向 Java 的契约框架。
- `Spring Cloud Zookeeper`: 基于 Apache Zookeeper 的服务注册和发现。
- `Spring Cloud Data Flow`: 在一个结构化的平台上，组成数据微服务。
- `Spring Cloud Kubernetes`: Spring Cloud 提供的针对 Kubernetes 的支持。
- `Spring Cloud Function`: 
- `Spring Cloud Task`: 短生命周期的微服务。

||Spring Cloud 第一代|Spring Cloud 第二代|
|:-|:-|:-|
|网关|spring-cloud-zuul（来源于 Netflix Zuul ，性能一般）|Spring Cloud Gateway|
|注册中心|spring-cloud-eureka（集成于 Netflix Eureka ，不再维护，Consul，ZK）|[阿里 Nacos](https://github.com/alibaba/nacos) ，拍拍贷 radar 等可选|
|配置中心|spring-cloud-config（自研，功能不足，国内使用其它配置中心替代）|[阿里 Nacos](https://github.com/alibaba/nacos) ，[携程 Apollo](https://github.com/ctripcorp/apollo) ，[随行付 Config Keeper](https://github.com/sxfad/config-keeper)|
|客户端软负载均衡|spring-cloud-ribbon（来源于 Netflix 集成，不支持 webFlux 的负载均衡）|[spring-cloud-loadbalancer](https://github.com/spring-cloud-incubator/spring-cloud-loadbalancer)|
|熔断器|spring-cloud-hystrix（来源于 Netflix 集成，不再开发新功能，进入维护状态）|[spring-cloud-r4j(Resilience4J)](https://github.com/spring-cloud-incubator/spring-cloud-r4j)，[阿里 Sentinel](https://github.com/alibaba/Sentinel)|

### 2.5 与 Spring Boot 的版本对应关系

|Spring Cloud 版本|Spring Boot 版本|
|:-|:-|
|`Hoxton`|2.2.x|
|`Greenwich`|2.1.x|
|`Finchley`|2.0.x|
|`Edgware`|1.5.x|
|`Dalston`|1.5.x|

---

- [Spring Cloud 教程合集](https://mp.weixin.qq.com/s/SBmcs2bxumhNz4kky1pl-A)（微信左下方**阅读全文**可直达）。
- Spring Cloud 教程合集示例代码：[https://github.com/cxy35/spring-cloud-samples](https://github.com/cxy35/spring-cloud-samples)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)