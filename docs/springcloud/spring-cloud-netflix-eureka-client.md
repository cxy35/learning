---
title: Spring Cloud Netflix Eureka Client 服务注册与发现
date: 2020-04-10 13:32:09
categories: Spring Cloud
tags: [Spring Cloud, Eureka]
toc: true
---
学习在 Spring Cloud 中使用 Eureka Client 实现服务注册与发现。
<!-- more -->

## 1 服务注册

服务注册就是把一个微服务注册到 Eureka Server 服务注册中心上，这样，当其他服务需要调用该服务时，只需要从 Eureka Server 上查询该服务的信息即可。

创建 Spring Boot 项目 `eureka-client-provider` ，作为我们的**服务提供者**，添加 `Web/Eureka Client` 依赖，如下：

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

项目创建成功后，修改 `application.properties` 配置文件，将 provider 注册到 Eureka Server 上，如下：

```properties
# 当前服务的名称
spring.application.name=provider
# 当前服务的端口
server.port=1113

# 服务注册中心地址
eureka.client.service-url.defaultZone=http://127.0.0.1:1111/eureka
```

接下来，启动 Eureka Server ，待服务注册中心启动成功后，再启动 provider ，两者都启动成功后，访问 [http://127.0.0.1:1111](http://127.0.0.1:1111) 可以看到 provider 的注册信息。

![](https://oscimg.oschina.net/oscnet/up-47e9eb05ac5e840641edd6d669dc55a6122.png)

---
 
最后在 provider 提供一个 hello 接口，用于后续服务消费者 consumer 来消费，如下：

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
 
## 2 服务消费

创建 Spring Boot 项目 `eureka-client-consumer` ，作为我们的**服务消费者**，添加 `Web/Eureka Client` 依赖，如下：

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

项目创建成功后，修改 `application.properties` 配置文件，将 consumer 注册到 Eureka Server 上，如下：

```properties
# 当前服务的名称
spring.application.name=consumer
# 当前服务的端口
server.port=1115

# 服务注册中心地址
eureka.client.service-url.defaultZone=http://127.0.0.1:1111/eureka
```

接下来，启动 consumer ，访问 [http://127.0.0.1:1111](http://127.0.0.1:1111) 可以看到 consumer 的注册信息。

![](https://oscimg.oschina.net/oscnet/up-94d2dcda13fa0a9a29d49ca56a2fdc57709.png)

---

下面开始演示在服务消费者 consumer 中新增测试接口，分别用两种 Http 请求工具（ `HttpURLConnection` 和 `RestTemplate` ）去实现服务调用，从而消费 provider 中提供的接口。

### 2.1 服务调用 - HttpURLConnection

1. 通过 HttpURLConnection（ JDK 自带，类似于 HttpClient ） 调用 provider 中的接口，写死 provider 的地址，整个调用过程不会涉及到 Eureka Server 。

```java
// 通过 HttpURLConnection（ JDK 自带，类似于 HttpClient ） 调用 provider 中的接口，写死 provider 的地址
@GetMapping("/testByHttpURLConnection")
public String testByHttpURLConnection() {
    HttpURLConnection con = null;
    try {
        URL url = new URL("http://127.0.0.1:1113/hello");
        con = (HttpURLConnection) url.openConnection();
        if (con.getResponseCode() == 200) {
            BufferedReader br = new BufferedReader(new InputStreamReader(con.getInputStream()));
            String s = br.readLine();
            br.close();
            return s;
        }
    } catch (MalformedURLException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
    return "error";
}
```

上述代码中 provider 和 consumer 高度绑定在一起，这个不符合微服务的思想。 consumer 要能够获取到 provider 这个接口的地址，他就需要去 Eureka Server 中查询，如果直接在 consumer 中写死 provider 地址，意味着这两个服务之间的耦合度就太高了，我们要降低耦合度。可以借助 Eureka Client 提供的 `DiscoveryClient` 工具，利用这个工具，我们可以根据**服务名**从 Eureka Server 上查询到一个服务的详细信息。

访问 [http://127.0.0.1:1115/testByHttpURLConnection](http://127.0.0.1:1115/testByHttpURLConnection) 完成测试。

---

2. 通过 HttpURLConnection（ JDK 自带，类似于 HttpClient ） 调用 provider 中的接口，通过 DiscoveryClient 动态获取 provider 的地址。

```java
@Autowired
DiscoveryClient discoveryClient;

// 通过 HttpURLConnection（ JDK 自带，类似于 HttpClient ） 调用 provider 中的接口，通过 DiscoveryClient 动态获取 provider 的地址
@GetMapping("/testByHttpURLConnection2")
public String testByHttpURLConnection2() {
    List<ServiceInstance> list = discoveryClient.getInstances("provider");
    ServiceInstance instance = list.get(0);
    String host = instance.getHost();
    int port = instance.getPort();
    StringBuffer sb = new StringBuffer();
    sb.append("http://")
            .append(host)
            .append(":")
            .append(port)
            .append("/hello");
    HttpURLConnection con = null;
    try {
        URL url = new URL(sb.toString());
        con = (HttpURLConnection) url.openConnection();
        if (con.getResponseCode() == 200) {
            BufferedReader br = new BufferedReader(new InputStreamReader(con.getInputStream()));
            String s = br.readLine();
            br.close();
            return s;
        }
    } catch (MalformedURLException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
    return "error";
}
```

注意： DiscoveryClient 查询到的服务列表是一个集合，因为服务在部署的过程中， provider 可能是集群化部署，集合中的每一项就是一个实例。

下面对 eureka-client-provider 进行打包，之后我们在命令行启动两个 provider 实例：
 
```bash
java -jar eureka-client-provider-0.0.1-SNAPSHOT.jar --server.port=1113
java -jar eureka-client-provider-0.0.1-SNAPSHOT.jar --server.port=1114
```

启动完成后，检查 Eureka Server 上，这两个 provider 是否成功注册上来。注册成功后，在 consumer 中再去调用 provider ，这样 DiscoveryClient 集合中，获取到的就不是一个实例了，而是两个实例，可以做客户端负载均衡。

访问 [http://127.0.0.1:1115/testByHttpURLConnection2](http://127.0.0.1:1115/testByHttpURLConnection2) 完成测试。

---

3. 通过 HttpURLConnection（ JDK 自带，类似于 HttpClient ） 调用 provider 中的接口，通过 DiscoveryClient 动态获取 provider 的地址，并手动实现客户端线性负载均衡。

```java
// 通过 HttpURLConnection（ JDK 自带，类似于 HttpClient ） 调用 provider 中的接口，通过 DiscoveryClient 动态获取 provider 的地址，并手动实现客户端线性负载均衡
int count = 0;
@GetMapping("/testByHttpURLConnection3")
public String testByHttpURLConnection3() {
    List<ServiceInstance> list = discoveryClient.getInstances("provider");
    ServiceInstance instance = list.get((count++) % list.size());
    String host = instance.getHost();
    int port = instance.getPort();
    StringBuffer sb = new StringBuffer();
    sb.append("http://")
            .append(host)
            .append(":")
            .append(port)
            .append("/hello");
    HttpURLConnection con = null;
    try {
        URL url = new URL(sb.toString());
        con = (HttpURLConnection) url.openConnection();
        if (con.getResponseCode() == 200) {
            BufferedReader br = new BufferedReader(new InputStreamReader(con.getInputStream()));
            String s = br.readLine();
            br.close();
            return s;
        }
    } catch (MalformedURLException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
    return "error";
}
```

访问 [http://127.0.0.1:1115/testByHttpURLConnection3](http://127.0.0.1:1115/testByHttpURLConnection3) 完成测试。

### 2.2 服务调用 - RestTemplate（推荐）

上面 HttpURLConnection 的调用过程有点繁琐，我们可以使用 Spring 提供的 RestTemplate 来实现。更多使用说明可参考 [Spring Cloud 中 RestTemplate 的使用说明](https://mp.weixin.qq.com/s/mJkqtckvWnxia4yOmZqS6w) 。

首先，在 consumer 的启动类中提供两个 RestTemplate 类型的 bean ，其中一个支持负载均衡，如下：

```java
@SpringBootApplication
public class EurekaClientConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaClientConsumerApplication.class, args);
    }

    @Bean
    RestTemplate restTemplate() {
        return new RestTemplate();
    }

    @Bean
    @LoadBalanced // 开启负载均衡
    RestTemplate restTemplateLoadBalanced() {
        return new RestTemplate();
    }
}
```

1. 【推荐】通过 RestTemplate 调用 provider 中的接口，通过 DiscoveryClient 动态获取 provider 的地址。

```java
// 【推荐】通过 RestTemplate 调用 provider 中的接口，通过 DiscoveryClient 动态获取 provider 的地址
@Autowired
@Qualifier("restTemplate")
RestTemplate restTemplate;

@GetMapping("/testByRestTemplate")
public String testByRestTemplate() {
    List<ServiceInstance> list = discoveryClient.getInstances("provider");
    ServiceInstance instance = list.get(0);
    String host = instance.getHost();
    int port = instance.getPort();
    StringBuffer sb = new StringBuffer();
    sb.append("http://")
            .append(host)
            .append(":")
            .append(port)
            .append("/hello");
    String s = restTemplate.getForObject(sb.toString(), String.class);
    return s;
}
```

访问 [http://127.0.0.1:1115/testByRestTemplate](http://127.0.0.1:1115/testByRestTemplate) 完成测试。

2. 【推荐】通过 RestTemplate 调用 provider 中的接口，通过 DiscoveryClient 动态获取 provider 的地址，并使用 @LoadBalanced 实现客户端负载均衡。

```java
// 【推荐】通过 RestTemplate 调用 provider 中的接口，通过 DiscoveryClient 动态获取 provider 的地址，并使用 @LoadBalanced 实现客户端负载均衡
@Autowired
@Qualifier("restTemplateLoadBalanced")
RestTemplate restTemplateLoadBalanced;

@GetMapping("/testByRestTemplate2")
public String testByRestTemplate2() {
    return restTemplateLoadBalanced.getForObject("http://provider/hello", String.class);
}
```

访问 [http://127.0.0.1:1115/testByRestTemplate](http://127.0.0.1:1115/testByRestTemplate) 完成测试。

---

- [Spring Cloud 教程合集](https://mp.weixin.qq.com/s/SBmcs2bxumhNz4kky1pl-A)（微信左下方**阅读全文**可直达）。
- Spring Cloud 教程合集示例代码：[https://github.com/cxy35/spring-cloud-samples](https://github.com/cxy35/spring-cloud-samples)
- 本文示例代码：[https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-eureka](https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-eureka)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)