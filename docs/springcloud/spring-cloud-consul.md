---
title: Spring Cloud Consul 服务注册与发现
date: 2020-04-15 14:48:24
categories: Spring Cloud
tags: [Spring Cloud, Consul]
toc: true
---
学习在 Spring Cloud 中使用 Consul 实现服务注册与发现，它是 Eureka 之外的另一种选择。
<!-- more -->

## 1 概述

在 Spring Cloud 中，大部分组件都有备选方案，例如服务注册中心，除了常见 `Eureka` 之外，还有 `Zookeeper` 和 `Consul` 等。 Consul 是 HashiCorp 公司推出来的开源产品，主要提供了**服务发现、服务隔离、服务配置**等功能。

相比于 Eureka 和 Zookeeper ，**Consul 配置更加一站式**，因为它内置了很多微服务常见的需求，比如：**服务发现与注册、分布式一致性协议实现、健康检查、键值对存储、多数据中心**等，我们不再需要借助第三方组件来实现这些功能。

## 2 安装

不同于 Eureka ， Consul 使用 Go 语言开发，所以，使用 Consul ，我们需要先 [下载](https://www.consul.io/downloads.html) 并安装软件，作为我们的 `Consul Server` ，即服务注册中心。

- Linux

```bash
# 下载
wget https://releases.hashicorp.com/consul/1.6.2/consul_1.6.2_linux_amd64.zip

# 解压，解压完成后，我们在当前目录下就可以看到 consul 文件
unzip consul_1.6.2_linux_amd64.zip

# 启动
./consul agent -dev -ui -node=consul-dev -client=127.0.0.1
```

- Windows

```bash
# 下载
# consul_1.7.2_windows_amd64.zip

# 启动
consul.exe agent -dev -ui -node=consul-dev -client=127.0.0.1
```

启动成功后，通过 [http://127.0.0.1:8500](http://127.0.0.1:8500) 访问 Consul 的后台管理页面，如下：

![](https://oscimg.oschina.net/oscnet/up-396eb62aa24f2625baf8435050c84318fe4.png)

## 3 使用

### 3.1 服务注册

创建 Spring Boot 项目 `consul-client-provider` ，作为我们的**服务提供者**，添加 `Web/Consul/Actuator` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-624d3b2e0be95ca43ab3de016ac3cb49b24.png)

最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-consul-discovery</artifactId>
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

项目创建成功后，修改 `application.properties` 配置文件，将 provider 注册到 Consul 服务上，如下：

```properties
spring.application.name=consul-client-provider
server.port=2000

# Consul 相关配置
spring.cloud.consul.host=127.0.0.1
spring.cloud.consul.port=8500
spring.cloud.consul.discovery.service-name=consul-client-provider
```

接着，在项目启动类上添加 `@EnableDiscoveryClient` 注解，开启服务发现的功能，如下：

```java
@SpringBootApplication
@EnableDiscoveryClient // 开启服务发现的功能
public class ConsulClientProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsulClientProviderApplication.class, args);
    }

}
```

接下来，启动项目，访问 [http://127.0.0.1:8500](http://127.0.0.1:8500) 可以看到 provider 的注册信息。

![](https://oscimg.oschina.net/oscnet/up-fbf76efc6b2ee5c8226ef75ee562def792c.png)

---

![](https://oscimg.oschina.net/oscnet/up-97c4b56ebe7377a7adcc25da8e71da732a3.png)

---

当然 provider 也可以集群化部署，下面对 consul-client-provider 进行打包，之后我们在命令行启动两个 provider 实例：
 
```bash
java -jar consul-client-provider-0.0.1-SNAPSHOT.jar --server.port=2000
java -jar consul-client-provider-0.0.1-SNAPSHOT.jar --server.port=2001
```

启动成功后，再去 Consul 后台管理页面，就可以看到有两个实例了：

![](https://oscimg.oschina.net/oscnet/up-f5a17cabfbd6b0fecd6fad7a4fee864fdb4.png)

---

最后在 provider 中提供一个 hello 接口，用于后续服务消费者 consumer 来消费，如下：

```java
@RestController
public class ProviderController {
    @Value("${server.port}")
    Integer port; // 支持启动多个实例，做负载均衡，用端口区分

    @GetMapping("/hello")
    public String hello() {
        return "hello cxy35: " + port;
    }
}
```

### 3.2 服务消费

创建 Spring Boot 项目 `consul-client-consumer` ，作为我们的**服务消费者**，添加 `Web/Consul/Actuator` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-624d3b2e0be95ca43ab3de016ac3cb49b24.png)

最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-consul-discovery</artifactId>
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

项目创建成功后，修改 `application.properties` 配置文件，将 consumer 注册到 Consul 服务上，如下：

```properties
spring.application.name=consul-client-consumer
server.port=2002

# Consul 相关配置
spring.cloud.consul.host=127.0.0.1
spring.cloud.consul.port=8500
spring.cloud.consul.discovery.service-name=consul-client-consumer

```

接着，在项目启动类上添加 `@EnableDiscoveryClient` 注解，开启服务发现的功能，并添加 RestTemplate ，如下：

```java
@SpringBootApplication
@EnableDiscoveryClient // 开启服务发现的功能
public class ConsulClientConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsulClientConsumerApplication.class, args);
    }

    @Bean
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

最后在 consumer 中新增测试接口，去实现服务调用，从而消费 provider 中提供的接口，如下：

```java
@RestController
public class ConsumerController {
    @Autowired
    LoadBalancerClient loadBalancerClient;
    @Autowired
    RestTemplate restTemplate;

    @GetMapping("/testConsul")
    public String testConsul() {
        ServiceInstance choose = loadBalancerClient.choose("consul-client-provider");
        System.out.println("服务地址：" + choose.getUri());
        System.out.println("服务名称：" + choose.getServiceId());
        String s = restTemplate.getForObject(choose.getUri() + "/hello", String.class);
        return s;
    }
}
```

我们通过 `LoadBalancerClient` 实例，可以获取要调用的 `ServiceInstance` 。获取到调用地址之后，再用 RestTemplate 去调用。

访问 [http://127.0.0.1:2002/testConsul](http://127.0.0.1:2002/testConsul) 完成测试，**这个请求自带负载均衡功能**。

---

- [Spring Cloud 教程合集](https://mp.weixin.qq.com/s/SBmcs2bxumhNz4kky1pl-A)（微信左下方**阅读全文**可直达）。
- Spring Cloud 教程合集示例代码：[https://github.com/cxy35/spring-cloud-samples](https://github.com/cxy35/spring-cloud-samples)
- 本文示例代码：[https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-consul](https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-consul)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)