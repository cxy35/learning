---
title: Spring Cloud OpenFeign 声明式服务调用
date: 2020-04-24 13:15:40
categories: Spring Cloud
tags: [Spring Cloud, OpenFeign]
toc: true
---
学习在 Spring Cloud 中使用 OpenFeign 实现声明式服务调用，包括简单调用、参数传递、继承特性、日志配置、数据压缩、服务降级/容错等功能。
<!-- more -->

## 1 概述

前面无论是基本调用，还是 Hystrix ，我们实际上都是通过手动调用 RestTemplate 来实现远程调用的。使用 RestTemplate 比较繁琐，每一个请求的参数、请求地址、返回数据类型不同，其他都是一样的，所以我们希望能够对请求进行简化，简化方案就是 OpenFeign 。

一开始这个组件叫 Feign/Netﬂix Feign ，但是 Netﬂix 中的组件，现在已经停止开源工作， OpenFeign 是 Spring Cloud 团队在 Netflix Feign 的基础上开发出来的**声明式服务调用**组件。关于 OpenFeign 组件的 Issue 见 [https://github.com/OpenFeign/feign/issues/373](https://github.com/OpenFeign/feign/issues/373) 。

## 2 准备工作

### 2.1 服务注册

创建 Spring Boot 项目 `openfeign-client-provider` ，作为我们的**服务提供者**，添加 `Web/Eureka Client` 依赖，如下：

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

项目创建成功后，修改 `application.properties` 配置文件，将 openfeign-client-provider 注册到 Eureka Server 上（服务注册中心使用 Eureka Server ），如下：

```properties
# 当前服务的名称
spring.application.name=openfeign-client-provider
# 当前服务的端口
server.port=4000

# 服务注册中心地址
eureka.client.service-url.defaultZone=http://127.0.0.1:1111/eureka
```

接下来，启动 Eureka Server ，待服务注册中心启动成功后，再启动 openfeign-client-provider ，两者都启动成功后，访问 [http://127.0.0.1:1111](http://127.0.0.1:1111) 可以看到 openfeign-client-provider 的注册信息。

![](https://oscimg.oschina.net/oscnet/up-0bf527e069fd08c8f28a97965c5ce743cd2.png)

---

当然 openfeign-client-provider 也可以集群化部署，下面对 openfeign-client-provider 进行打包，之后我们在命令行启动两个 provider 实例：
 
```bash
java -jar openfeign-client-provider-0.0.1-SNAPSHOT.jar --server.port=4000
java -jar openfeign-client-provider-0.0.1-SNAPSHOT.jar --server.port=4001
```

---

最后在 openfeign-client-provider 提供一个 hello 接口，用于后续服务消费者 openfeign-client-consumer 来消费，如下：

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

### 2.2 服务消费

创建 Spring Boot 项目 `openfeign-client-consumer` ，作为我们的**服务消费者**，添加 `Web/Eureka Client/OpenFeign` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-9c7f2b43ba87fb47af110d4acfaba33da5d.png)

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
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
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

项目创建成功后，修改 `application.properties` 配置文件，将 openfeign-client-consumer 注册到 Eureka Server 上（服务注册中心使用 Eureka Server ），如下：

```properties
# 当前服务的名称
spring.application.name=openfeign-client-consumer
# 当前服务的端口
server.port=4002

# 服务注册中心地址
eureka.client.service-url.defaultZone=http://127.0.0.1:1111/eureka
```

接着，在项目启动类上添加 `@EnableFeignClients` 注解，开启 OpenFeign 功能，如下：

```java
@SpringBootApplication
@EnableFeignClients // 开启 OpenFeign 功能
public class OpenfeignClientConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(OpenfeignClientConsumerApplication.class, args);
    }

}
```

接下来，启动 openfeign-client-consumer ，访问 [http://127.0.0.1:1111](http://127.0.0.1:1111) 可以看到 openfeign-client-consumer 的注册信息。

![](https://oscimg.oschina.net/oscnet/up-233bf91a5f2279fb7896c0358796c344eb9.png)

---

最后在 openfeign-client-consumer 中新增测试业务类和接口，去实现服务调用，从而消费 openfeign-client-provider 中提供的接口，如下：

> 约定：本文中的服务调用失败（测试服务降级/容错），可以采用关闭某个 openfeign-client-provider 来模拟，短时间内会报错（因为 provider 地址会缓存 consumer 上一段时间），从而达到我们的目的。

## 3 简单调用

新建测试业务类 `ConsumerService` ，如下：

```java
@FeignClient("openfeign-client-provider")
public interface ConsumerService {

    @GetMapping("/hello")
    String hello(); // 这里的方法名无所谓，随意取
}
```

---

新建测试接口 `ConsumerController` ，如下：

```java
@RestController
public class ConsumerController {
    @Autowired
    ConsumerService consumerService;

    @GetMapping("/hello")
    public String hello() {
        return consumerService.hello();
    }
}
```

访问 [http://127.0.0.1:4002/hello](http://127.0.0.1:4002/hello) 完成测试。

## 4 参数传递

OpenFeign 中的请求参数传递与普通请求参数传递的区别如下：

1. 参数一定要绑定参数名。
2. 如果通过 header 来传递参数，一定记得中文要转码。

修改 `ProviderController` ，增加测试接口，如下：

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
        System.out.println(new Date());
        return "hello " + name + ": " + port;
    }

    @PostMapping("/user")
    public User addUser(@RequestBody User user) {
        return user;
    }

    @DeleteMapping("/user/{id}")
    public void deleteUser(@PathVariable Integer id) {
        System.out.println(id);
    }

    @GetMapping("/user")
    public void getUserByName(@RequestHeader String name) throws UnsupportedEncodingException {
        System.out.println(URLDecoder.decode(name, "UTF-8"));
    }
}
```

---

修改 `ConsumerService` ，增加各种类型的测试接口，如下：

```java
@FeignClient("openfeign-client-provider")
public interface ConsumerService {

    @GetMapping("/hello")
    String hello(); // 这里的方法名无所谓，随意取

    @GetMapping("/hello2")
    String hello2(@RequestParam("name") String name);

    @PostMapping("/user")
    User addUser(@RequestBody User user);

    @DeleteMapping("/user/{id}")
    void deleteUser(@PathVariable("id") Integer id);

    @GetMapping("/user")
    void getUserByName(@RequestHeader("name") String name);
}
```

**注意，凡是 key/value 形式的参数，一定要标记参数的名称。**

---

修改 `ConsumerController` ，增加测试接口，如下：

```java
@RestController
public class ConsumerController {
    @Autowired
    ConsumerService consumerService;

    @GetMapping("/hello")
    public String hello() {
        return consumerService.hello();
    }

    @GetMapping("/testOpenFeign")
    public String testOpenFeign() throws UnsupportedEncodingException {
        String s = consumerService.hello();

        String s2 = consumerService.hello2("程序员35");
        System.out.println(s2);

        User user = new User();
        user.setId(1);
        user.setUsername("cxy35");
        user.setPassword("123");

        User u = consumerService.addUser(user);
        System.out.println(u);

        consumerService.deleteUser(1);

        consumerService.getUserByName(URLEncoder.encode("程序员35", "UTF-8"));

        return s;
    }
}
```

访问 [http://127.0.0.1:4002/testOpenFeign](http://127.0.0.1:4002/testOpenFeign) 完成测试。

**注意：放在 header 中的中文参数，一定要编码之后传递。**

## 5 继承特性

修改 spring-cloud-common 模块，增加一个公共的接口，给 openfeign-client-provider 和 openfeign-client-consumer 使用。但是由于这个模块要用到 Spring MVC 的东西，因此添加 Web 依赖，最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.2.6.RELEASE</version>
    </dependency>
</dependencies>
```

在 spring-cloud-common 中新增 `OpenFeignService` 接口，**里面的内容就是上文中 ConsumerService 接口的内容**，如下：

```java
public interface OpenFeignService {
    @GetMapping("/hello")
    String hello(); // 这里的方法名无所谓，随意取

    @GetMapping("/hello2")
    String hello2(@RequestParam("name") String name);

    @PostMapping("/user")
    User addUser(@RequestBody User user);

    @DeleteMapping("/user/{id}")
    void deleteUser(@PathVariable("id") Integer id);

    @GetMapping("/user")
    void getUserByName(@RequestHeader("name") String name) throws UnsupportedEncodingException;
}
```

---

在 openfeign-client-provider 中新增 `ProviderController2` 类，实现 `OpenFeignService` 接口，并实现全部方法，如下：

```java
@RestController
public class ProviderController2 implements OpenFeignService {
    @Value("${server.port}")
    Integer port; // 支持启动多个实例，做负载均衡，用端口区分

    @Override
    public String hello() {
        return "hello2 cxy35: " + port;
    }

    @Override
    public String hello2(String name) {
        System.out.println(new Date());
        return "hello2 " + name + ": " + port;
    }

    @Override
    public User addUser(@RequestBody User user) {
        return user;
    }

    @Override
    public void deleteUser(@PathVariable Integer id) {
        System.out.println(id);
    }

    @Override
    public void getUserByName(@RequestHeader String name) throws UnsupportedEncodingException {
        System.out.println(URLDecoder.decode(name, "UTF-8"));
    }
}
```

修改 `ProviderController` ，并注释掉 @RestController ，避免与 ProviderController2 重复，导致启动报错。

```java
// @RestController
public class ProviderController {
  ......
}
```

---

在 openfeign-client-consumer 中新增 `ConsumerService2` 接口，继承 `OpenFeignService` 接口，如下：

```java
@FeignClient("openfeign-client-provider")
public interface ConsumerService2 extends OpenFeignService {
}
```

修改 `ConsumerService` ，并注释掉 @FeignClient ，避免与 ConsumerService2 重复，导致启动报错。

```java
// @FeignClient("openfeign-client-provider")
public interface ConsumerService {
  ......
}
```

修改 `ConsumerController` ，换成新的 ConsumerService ，避免找不到 ConsumerService ，导致启动报错

```java
// @Autowired
// ConsumerService consumerService;
@Autowired
ConsumerService2 consumerService;
```

访问 [http://127.0.0.1:4002/testOpenFeign](http://127.0.0.1:4002/testOpenFeign) 完成测试。

---

关于继承特性：

1. 使用继承特性，代码简洁明了不易出错。服务端和消费端的代码统一，一改俱改，不易出错。这是优点也是缺点，这样会提高服务端和消费端的耦合度。
2. 上文中所讲的参数传递，在使用了继承之后，依然不变，参数该怎么传还是怎么传。

## 6 日志配置

OpenFeign 中，我们可以通过配置日志，来查看整个请求的调用过程。日志级别一共分为四种：

1. NONE：不开启日志，默认就是这个
2. BASIC：记录请求方法、URL、响应状态码、执行时间
3. HEADERS：在 BASIC 的基础上，加载请求/响应头
4. FULL：在 HEADERS 基础上，再增加 Body 以及请求元数据。

配置方式如下：

```java
@SpringBootApplication
@EnableFeignClients // 开启 OpenFeign 功能
public class OpenfeignClientConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(OpenfeignClientConsumerApplication.class, args);
    }

    @Bean
    Logger.Level loggerLevel() {
        return Logger.Level.FULL;
    }

}
```

另外，还要在 application.properties 中配置日志级别：

```xml
# 配置 OpenFeign 的日志级别
logging.level.com.cxy35.sample.springcloud.openfeign.client.consumer.service=debug
```

## 7 数据压缩

修改配置文件，如下：

```xml
# 开启请求的数据压缩
feign.compression.request.enabled=true
# 开启响应的数据压缩
feign.compression.response.enabled=true
# 压缩的数据类型，默认如下
feign.compression.request.mime-types=text/html,application/xml,application/json
# 压缩的数据下限，默认 2048 表示当要传输的数据大于 2048 时，才会进行数据压缩
feign.compression.request.min-request-size=2048
```

## 8 服务降级/容错

Hystrix 中的服务降级/容错等功能，在 OpenFeign 中一样要使用，有两种方式。

首先，在 `application.properties` 中开启 Hystrix 。

```xml
# 开启 Hystrix
feign.hystrix.enabled=true
```

1. fallback

新增 `ConsumerService2Fallback` 类，实现 `ConsumerService2` 接口，实现对应服务降级的方法，如下：

```java
@Component
@RequestMapping("/abc") // 防止请求地址重复，可随意定义
public class ConsumerService2Fallback implements ConsumerService2 {
    @Override
    public String hello() {
        return "error-fallback";
    }

    @Override
    public String hello2(String name) {
        return "error2-fallback";
    }

    @Override
    public User addUser(User user) {
        return null;
    }

    @Override
    public void deleteUser(Integer id) {
    }

    @Override
    public void getUserByName(String name) throws UnsupportedEncodingException {
    }
}
```

接着，在 `ConsumerService2` 中配置这个服务降级类，如下：

```java
// @FeignClient("openfeign-client-provider")
@FeignClient(value = "openfeign-client-provider", fallback = ConsumerService2Fallback.class)
public interface ConsumerService2 extends OpenFeignService {
}
```

最后，关闭 openfeign-client-provider ，模拟服务调用失败，访问 [http://127.0.0.1:4002/hello](http://127.0.0.1:4002/hello) 完成测试。

---

2. fallbackFactory

新增 `ConsumerService2FallbackFactory` 类，实现 `ConsumerService2` 接口，实现对应服务降级的方法，如下：

```java
@Component
public class ConsumerService2FallbackFactory implements FallbackFactory<ConsumerService2> {
    @Override
    public ConsumerService2 create(Throwable throwable) {
        return new ConsumerService2() {
            @Override
            public String hello() {
                return "error-fallbackFactory";
            }

            @Override
            public String hello2(String name) {
                return "error2-fallbackFactory";
            }

            @Override
            public User addUser(User user) {
                return null;
            }

            @Override
            public void deleteUser(Integer id) {
            }

            @Override
            public void getUserByName(String name) throws UnsupportedEncodingException {
            }
        };
    }
}
```

接着，在 `ConsumerService2` 中配置这个服务降级类，如下：

```java
// @FeignClient("openfeign-client-provider")
// @FeignClient(value = "openfeign-client-provider", fallback = ConsumerService2Fallback.class)
@FeignClient(value = "openfeign-client-provider", fallbackFactory = ConsumerService2FallbackFactory.class)
public interface ConsumerService2 extends OpenFeignService {
}
```

最后，关闭 openfeign-client-provider ，模拟服务调用失败，访问 [http://127.0.0.1:4002/hello](http://127.0.0.1:4002/hello) 完成测试。

---

- [Spring Cloud 教程合集](https://mp.weixin.qq.com/s/SBmcs2bxumhNz4kky1pl-A)（微信左下方**阅读全文**可直达）。
- Spring Cloud 教程合集示例代码：[https://github.com/cxy35/spring-cloud-samples](https://github.com/cxy35/spring-cloud-samples)
- 本文示例代码：[https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-openfeign](https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-openfeign)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)