学习在 Spring Cloud 中使用 Zuul 实现服务网关，包括基本使用、请求过滤、忽略路径、前缀等功能。它是 Netflix 家族成员之一。
<!-- more -->

## 1 概述

由于每一个微服务的地址都有可能发生变化，无法直接对外公布这些服务地址，基于安全以及高内聚低耦合等设计，我们有必要将内部系统和外部系统做一个切割。

一个专门用来处理外部请求的组件，就是服务网关，常用功能：

- 权限问题统一处理
- 数据剪裁和聚合
- 简化客户端的调用
- 可以针对不同的客户端提供不同的网关支持

在 Spring Cloud 中，网关主要有两种实现方案： `Zuul` 和 `Spring Cloud Gateway` 。

Zuul 是 Netﬂix 公司提供的网关服务，主要有如下功能：

- 权限控制，可以做认证和授权
- 监控
- 动态路由
- 负载均衡
- 静态资源处理

Zuul 中的功能基本上都是基于**过滤器**来实现，它的过滤器有几种不同的类型：

- PRE
- ROUTING
- POST
- ERROR

## 2 准备工作

### 2.1 服务注册

创建 Spring Boot 项目 `zuul-client-provider` ，作为我们的**服务提供者**，添加 `Web/Eureka Client` 依赖，如下：

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

项目创建成功后，修改 `application.properties` 配置文件，将 zuul-client-provider 注册到 Eureka Server 上（服务注册中心使用 Eureka Server ），如下：

```properties
# 当前服务的名称
spring.application.name=zuul-client-provider
# 当前服务的端口
server.port=6000

# 服务注册中心地址
eureka.client.service-url.defaultZone=http://127.0.0.1:1111/eureka
```

接下来，启动 Eureka Server ，待服务注册中心启动成功后，再启动 zuul-client-provider ，两者都启动成功后，访问 [http://127.0.0.1:1111](http://127.0.0.1:1111) 可以看到 zuul-client-provider 的注册信息。

---

当然 zuul-client-provider 也可以集群化部署，下面对 zuul-client-provider 进行打包，之后我们在命令行启动两个 provider 实例：
 
```bash
java -jar zuul-client-provider-0.0.1-SNAPSHOT.jar --server.port=6000
java -jar zuul-client-provider-0.0.1-SNAPSHOT.jar --server.port=6001
```

---

最后在 zuul-client-provider 提供一个 hello 接口，用于后续服务消费者 zuul-client-consumer 来消费，如下：

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

创建 Spring Boot 项目 `zuul-client-consumer` ，作为我们的**服务消费者**，添加 `Web/Eureka Client/Zuul` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-553a8a9b81a1973ce4dc755aa06410b4ce9.png)

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
      <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
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

项目创建成功后，修改 `application.properties` 配置文件，将 zuul-client-consumer 注册到 Eureka Server 上（服务注册中心使用 Eureka Server ），如下：

```properties
# 当前服务的名称
spring.application.name=zuul-client-consumer
# 当前服务的端口
server.port=6002

# 服务注册中心地址
eureka.client.service-url.defaultZone=http://127.0.0.1:1111/eureka
```

接着，在项目启动类上添加 `@EnableZuulProxy` 注解，开启网关代理，如下：

```java
@SpringBootApplication
@EnableZuulProxy // 开启网关代理
public class ZuulClientConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ZuulClientConsumerApplication.class, args);
    }

}
```

接下来，启动 zuul-client-consumer ，访问 [http://127.0.0.1:1111](http://127.0.0.1:1111) 可以看到 zuul-client-consumer 的注册信息。

## 3 基本使用

配置完成后，访问 [http://127.0.0.1:6002/zuul-client-provider/hello](http://127.0.0.1:6002/zuul-client-provider/hello) ，会自动通过 Zuul 的代理访问到 zuul-client-provider 中对应的的接口，无需知道其真实的地址。在这个访问地址中， **zuul-client-provider 就是要访问的服务名称， /hello 则是要访问的服务接口**。

当然， Zuul 中的路由规则也可以自己配置，如下：

```properties
# Zuul 中的路由规则配置
# zuul.routes.cxy35.path=/cxy35/**
# zuul.routes.cxy35.service-id=zuul-client-provider

# 简化配置
zuul.routes.zuul-client-provider=/cxy35/**
```

上面配置表示满足 `/cxy35/**` 这个匹配规则的请求，将被转发到 zuul-client-provider 实例上。比如：[http://127.0.0.1:6002/cxy35/hello](http://127.0.0.1:6002/cxy35/hello)

## 4 请求过滤

对于来自客户端的请求，可以在 Zuul 中进行预处理，例如**权限判断**等。

在 `zuul-client-consumer` 中定义一个简单的权限过滤器，如下：

```java
@Component
public class PermissionFilter extends ZuulFilter {
    /**
     * 过滤器类型，权限判断一般是 pre
     *
     * @return
     */
    @Override
    public String filterType() {
        return "pre";
    }

    /**
     * 过滤器优先级
     *
     * @return
     */
    @Override
    public int filterOrder() {
        return 0;
    }

    /**
     * 是否过滤
     *
     * @return
     */


    @Override
    public boolean shouldFilter() {
        return true;
    }

    /**
     * 核心的过滤逻辑写在这里
     *
     * @return 这个方法虽然有返回值，但是这个返回值目前无所谓
     * @throws ZuulException
     */
    @Override
    public Object run() throws ZuulException {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();//获取当前请求
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        if (!"cxy35".equals(username) || !"123456".equals(password)) {
            //如果请求条件不满足的话，直接从这里给出响应
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(401);
            ctx.addZuulResponseHeader("content-type", "text/html;charset=utf-8");
            ctx.setResponseBody("非法访问，无权限");
        }
        return null;
    }
}
```

重启 zuul-client-consumer ，接下来，发送请求必须带上 username 和 password 参数，否则请求不通过，比如：[http://127.0.0.1:6002/cxy35/hello?username=cxy35&password=123456](http://127.0.0.1:6002/cxy35/hello?username=cxy35&password=123456)

## 5 其他配置

### 5.1 匹配规则

例如有两个服务，一个叫 provider ，另一个叫 provider-hello ，在做路由规则设置时，假如出现了如下配置：

```properties
zuul.routes.provider=/provider/**

zuul.routes.provider-hello=/provider/hello/**
```

此时，如果访问一个地址：`http://127.0.0.1:6002/provider/hello/123` ，会出现冲突。实际上，这个地址是希望和 provider-hello 这个服务匹配的，这个时候，只需要把配置文件改为 yml 格式就可以了，**因为 yml 格式的配置是有序的**。

### 5.2 忽略路径

默认情况下，Zuul 注册到 Eureka 上之后， Eureka 上的所有注册服务都会被自动代理。如果不想给某一个服务做代理，可以忽略该服务，配置如下：

```properties
zuul.ignored-services=provider2
```

上面这个配置表示忽略 provider2 服务，此时就不会自动代理 provider2 服务了。

---

当然，也可以忽略某一类地址，配置如下：

```properties
zuul.ignored-patterns=/**/hello/**
```

这个表示请求路径中如果包含 hello ，则不做代理。

### 5.3 前缀

可以统一给路由加前缀，配置如下：

```properties
zuul.prefix=/cxy35
```

这样，以后所有的请求地址自动多了前缀 `/cxy35` 。

---

- [Spring Cloud 教程合集](https://mp.weixin.qq.com/s/SBmcs2bxumhNz4kky1pl-A)（微信左下方**阅读全文**可直达）。
- Spring Cloud 教程合集示例代码：[https://github.com/cxy35/spring-cloud-samples](https://github.com/cxy35/spring-cloud-samples)
- 本文示例代码：[https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-netflix-zuul](https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-netflix-zuul)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.top](https://cxy35.top)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)