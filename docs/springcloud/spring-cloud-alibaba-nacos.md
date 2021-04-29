学习在 Spring Cloud 中使用 Nacos 实现服务配置中心和注册中心，类似 Spring Cloud Config 和 Spring Cloud Netflix Eureka 提供的功能。
<!-- more -->

## 1 概述

Spring Cloud Alibaba 是阿里巴巴提供的一套微服务开发一站式解决方案。

主要提供的功能：

- 分布式配置中心
- 服务注册与发现
- 服务限流降级
- 消息驱动
- 分布式事务
- 阿里云对象存储（绑定阿里云）
- 阿里云短信（绑定阿里云）

提供的组件：

- Nacos：主要提供了服务动态配置、服务及元数据管理、服务注册与发现、动态 DNS 服务。
- Sentinel：熔断、限流等。

优势：

1. 中文文档。
2. 没有另起炉灶，可以方便的集成到现有项目中。
3. 阿里本身在高并发、高性能上的经验，让我们有理由相信这些组件足够可靠。

## 2 安装

下载地址：[https://github.com/alibaba/nacos/releases](https://github.com/alibaba/nacos/releases)

解压后启动：

- Windows：在 bin 目录下执行 `startup.cmd -m standalone`
- Linux：在 bin 目录下执行 `sh startup.sh -m standalone`

> 注意：需要 java 和 javac 两个命令，可以先测试下。

Nacos 启动成功后，访问 [http://127.0.0.1:8848/nacos](http://127.0.0.1:8848/nacos) 就能看到后台页面。如果有登录页面，登录的默认用户名/密码都是 `nacos` 。

![](https://oscimg.oschina.net/oscnet/up-2f6005ad5daad773ec8b451de7e0e1472c1.png)

## 3 配置中心

Nacos 做配置中心，可以代替 Spring Cloud Config 。

创建 Spring Boot 项目 `nacos-config` ，添加 `Web/Nacos Configuration` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-87c8beac76e29203b8d3c1bacce44f60c07.png)

最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
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

首先，在 Nacos 服务端后台配置，页面上依次点击 `配置管理 -> 配置列表 -> +` ，这里主要配置三个东西，Data ID（nacos-config.properties）、Group 以及要配置的内容。其中 `Data ID` 的格式为 `${prefix}-${spring.profile.active}.${file-extension}` ：

- ${preﬁx}：默认为 spring.application.name 的值，如 `nacos-config` 。
- ${spring.proﬁle.active}：表示项目当前所处的环境，如 `dev/test/prod` 。
- ${ﬁle-extension}：表示配置文件的扩展名，如 `properties` 。

![](https://oscimg.oschina.net/oscnet/up-34483e3e52269f7e8cd922689d62a614fb8.png)

然后，新建 `bootstrap.properties` 配置文件，配置 Nacos 相关信息：

```properties
spring.application.name=nacos-config
server.port=8080

spring.cloud.nacos.config.server-addr=127.0.0.1:8848
spring.cloud.nacos.config.file-extension=properties
```

---

最后，提供一个 hello 接口，注意要添加 `@RefreshScope` 注解。

```java
@RestController
@RefreshScope
public class HelloController {
    @Value("${name}")
    String name;

    @GetMapping("/hello")
    public String hello() {
        return "hello : " + name;
    }
}
```

启动项目，访问 [http://127.0.0.1:8080/hello](http://127.0.0.1:8080/hello) 可以正常获取到 Nacos 服务端后台配置的配置文件中的属性值。**在 Nacos 后台修改属性值，保存后可以实时生效。**

## 4 注册中心
  
Nacos 做注册中心，可以代替 Spring Cloud Netflix Eureka 。

### 4.2 服务注册

创建 Spring Boot 项目 `nacos-client-provider` ，作为我们的**服务提供者**，添加 `Web/Nacos Service Discovery` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-08afcd53ef93bd0411ad7962a7031db03b5.png)

最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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

项目创建成功后，修改 `application.properties` 配置文件，如下：

```properties
spring.application.name=nacos-client-provider
server.port=9000

spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
```

然后，启动项目，在 Nacos 后台页面上可以看到这个实例。

![](https://oscimg.oschina.net/oscnet/up-ba056e8a3a574445826b305b1d7398a822a.png)

---

当然 provider 也可以集群化部署，下面对 nacos-client-provider 进行打包，之后我们在命令行启动两个 provider 实例：
 
```bash
java -jar nacos-client-provider-0.0.1-SNAPSHOT.jar --server.port=9000
java -jar nacos-client-provider-0.0.1-SNAPSHOT.jar --server.port=9001
```

启动成功后，在 Nacos 后台页面上可以看到这 2 个实例。

---

最后，在 provider 提供一个 hello 接口，用于后续服务消费者 consumer 来消费，如下：

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

### 4.2 服务消费

创建 Spring Boot 项目 `nacos-client-consumer` ，作为我们的**服务消费者**，添加 `Web/Nacos Service Discovery` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-08afcd53ef93bd0411ad7962a7031db03b5.png)

最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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

项目创建成功后，修改 `application.properties` 配置文件，如下：

```properties
spring.application.name=nacos-client-consumer
server.port=9002

spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
```

然后，启动项目，在 Nacos 后台页面上可以看到这个实例。

![](https://oscimg.oschina.net/oscnet/up-c21431c13795ca6aaa45abf3e90e2ef83c7.png)

接着，在项目启动类中添加 RestTemplate ，如下：

```java
@SpringBootApplication
public class NacosClientConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(NacosClientConsumerApplication.class, args);
    }

    @Bean
    @LoadBalanced // 开启负载均衡
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

---

最后在 consumer 中新增测试接口，去实现服务调用，从而消费 provider 中提供的接口，如下：

```java
@RestController
public class ConsumerController {
    @Autowired
    RestTemplate restTemplate;

    @GetMapping("/hello")
    public String hello() {
        return restTemplate.getForObject("http://nacos-client-provider/hello", String.class);
    }
}
```

访问 [http://127.0.0.1:9002/hello](http://127.0.0.1:9002/hello) 完成测试。

---

- [Spring Cloud 教程合集](https://mp.weixin.qq.com/s/SBmcs2bxumhNz4kky1pl-A)（微信左下方**阅读全文**可直达）。
- Spring Cloud 教程合集示例代码：[https://github.com/cxy35/spring-cloud-samples](https://github.com/cxy35/spring-cloud-samples)
- 本文示例代码：[https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-alibaba-nacos](https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-alibaba-nacos)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)