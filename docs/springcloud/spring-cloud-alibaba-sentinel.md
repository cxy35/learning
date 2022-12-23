学习在 Spring Cloud 中使用 Sentinel 实现断路器，类似 Spring Cloud Netflix Hystrix ，包括实时监控、簇点链路、流控、降级等功能。 Sentinel 提供的功能更强大，使用更方便，可以替代 Hystrix ，还能结合 Nacos 中的配置中心一起使用。
<!-- more -->

## 1 概述

Sentinel，中文翻译为哨兵，是为微服务提供流量控制、熔断降级的功能，它和 Hystrix 提供的功能一样，可以有效的解决微服务调用产生的雪崩“”效应，为微服务系统提供了稳定性的解决方案。随着 Hystrxi 进入了维护期，不再提供新功能，Sentinel 是一个不错的替代方案。通常情况，Hystrix 采用线程池对服务的调用进行隔离，Sentinel 才用了用户线程对接口进行隔离，二者相比，Hystrxi 是服务级别的隔离，Sentinel 提供了接口级别的隔离，Sentinel 隔离级别更加精细，另外 Sentinel 直接使用用户线程进行限制，相比 Hystrix 的线程池隔离，减少了线程切换的开销。另外 Sentinel 的 DashBoard 提供了在线更改限流规则的配置，也更加的优化。

从官方文档的介绍，Sentinel 具有以下特征:

- 丰富的应用场景：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、实时熔断下游不可用应用等。
- 完备的实时监控：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
- 广泛的开源生态：Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
- 完善的 SPI 扩展点：Sentinel 提供简单易用、完善的 SPI 扩展点。您可以通过实现扩展点，快速的定制逻辑。例如定制规则管理、适配数据源等。

Sentinel 整体上可以分为两个核心部分：**核心库和控制台**。

## 2 安装

首先下载控制台 jar 包，这是一个 Spring Boot 工程，下载好之后，直接启动。

下载地址：[https://github.com/alibaba/Sentinel/releases/download/1.7.2/sentinel-dashboard-1.7.2.jar](https://github.com/alibaba/Sentinel/releases/download/1.7.2/sentinel-dashboard-1.7.2.jar)

Sentinel 启动成功后，访问 [http://127.0.0.1:8080](http://127.0.0.1:8080) 就能看到后台页面了，默认用户名/密码都是 `sentinel` 。

![](https://oscimg.oschina.net/oscnet/up-80423f1a00b9e4428a4cfe533562914cec6.png)

## 3 基本使用

创建 Spring Boot 项目 `spring-cloud-alibaba-sentinel` ，添加 `Web/Sentinel` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-d3e39b6d9bd49a1f4f60b8a2069675f9e34.png)

最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
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

项目创建成功后，修改 `application.properties` 配置文件，配置 Sentinel 控制台地址，如下：

```properties
spring.application.name=spring-cloud-alibaba-sentinel
server.port=8081

# Sentinel 控制台地址
spring.cloud.sentinel.transport.dashboard=127.0.0.1:8080
```

---

接着，提供一个 hello 接口，如下：

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello sentinel";
    }
}
```

然后，在测试类中增加测试代码，如下：

```java
@Test
void contextLoads() {
    RestTemplate restTemplate = new RestTemplate();
    for (int i = 0; i < 15; i++) {
        String s = restTemplate.getForObject("http://127.0.0.1:8081/hello", String.class);
        System.out.println(s + ":" + new Date());
    }
}
```

---

启动项目之后，访问 [http://127.0.0.1:8081/hello](http://127.0.0.1:8081/hello) ，之后，在 Sentinel 控制台页面上就能看到这个应用及 `/hello` 接口了，下面新增流控配置，**限制每 1 秒中响应 5 个请求**：

![](https://oscimg.oschina.net/oscnet/up-bf9d8225d95ddb5e39944953aec28ac6bad.png)

![](https://oscimg.oschina.net/oscnet/up-d3c5b4b99d5cf0b0b7c81002fa495dbed4f.png)

最后，执行测试类中的方法，结果如下：

![](https://oscimg.oschina.net/oscnet/up-3372504418f41e6bcdcc7e3ae1f6e1749ff.png)

## 4 结合 Nacos 中的配置中心使用

Sentinel 还能结合 Nacos 中的配置中心一起使用。

首先，在 pom.xml 中增加 `Nacos Configuration 和 sentinel-datasource-nacos` 依赖，如下：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
    <version>1.7.1</version>
</dependency>
```

修改 `application.properties` 配置文件，增加 Nacos 相关配置信息，如下：

```properties
spring.application.name=spring-cloud-alibaba-sentinel
server.port=8081

# Sentinel 控制台地址
spring.cloud.sentinel.transport.dashboard=127.0.0.1:8080

# 结合 Nacos 中的配置中心使用
spring.cloud.sentinel.datasource.ds.nacos.server-addr=127.0.0.1:8848
spring.cloud.sentinel.datasource.ds.nacos.data-id=spring-cloud-alibaba-sentinel
spring.cloud.sentinel.datasource.ds.nacos.group-id=DEFAULT_GROUP
spring.cloud.sentinel.datasource.ds.nacos.rule-type=flow
```

接着，新建 `bootstrap.properties` 配置文件，配置 Nacos 相关信息：

```properties
spring.cloud.nacos.config.server-addr=127.0.0.1:8848
```

---

然后，在 Nacos 后台增加配置（注意 `Data Id/GROUP` 要与上述配置文件中的配置对应上）， **JSON 格式**，内容其实就是对应前面章节 Sentinel 控制台页面上配置的流控规则。

![](https://oscimg.oschina.net/oscnet/up-ffe3e60b4db1123b76a0e8612beb7cac8b2.png)

```
[
    {
        "resource":"/hello", // 资源名
        "limitApp":"default", // 针对来源
        "grade":1, // 阈值类型=QPS
        "count":5, // 单机阈值
        "clusterMode":false, // 是否集群
        "strategy":0, // 流控模式=直接
        "controlBehavior":2 // 流控效果=排队等待
    }
]
```

**最后，重启项目，访问 http://127.0.0.1:8081/hello ，之后， Sentinel 会根据 Nacos 后台的 spring-cloud-alibaba-sentinel 这项配置，自动在 Sentinel 控制台生成一条对应的流控规则**。

![](https://oscimg.oschina.net/oscnet/up-04c549bef3e0d4f2e95fdbd7983d604bf85.png)

测试结果如下：

![](https://oscimg.oschina.net/oscnet/up-2e2393af3d9cbfa2a75cd1008ca27ddebe9.png)

---

- [Spring Cloud 教程合集](https://mp.weixin.qq.com/s/SBmcs2bxumhNz4kky1pl-A)（微信左下方**阅读全文**可直达）。
- Spring Cloud 教程合集示例代码：[https://github.com/cxy35/spring-cloud-samples](https://github.com/cxy35/spring-cloud-samples)
- 本文示例代码：[https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-alibaba-sentinel](https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-alibaba-sentinel)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.top](https://cxy35.top)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)