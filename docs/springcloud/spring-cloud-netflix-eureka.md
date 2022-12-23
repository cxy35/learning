Eureka 是 Spring Cloud 中的服务注册中心，类似于 Dubbo 中的 Zookeeper ，它是 Netflix 家族成员之一。本文学习 Eureka 简介、搭建 Eureka 注册中心、搭建 Eureka 客户端等。
<!-- more -->

## 1 注册中心

什么是注册中心，我们为什么需要注册中心？我们首先来看一个传统的单体应用：

![](https://oscimg.oschina.net/oscnet/up-5a6a0a6cd084e644ff2a29b7ab774c13aef.png)

在单体应用中，所有的业务都集中在一个项目中，当用户从浏览器发起请求时，直接由前端发起请求给后端，后端调用业务逻辑，给前端请求做出响应，完成一次调用。整个调用过程是一条直线，不需要服务之间的中转，所以没有必要引入注册中心。

随着公司项目越来越大，我们会将系统进行拆分，例如一个电商项目，可以拆分为订单模块、物流模块、支付模块、 CMS 模块等。这样，当用户发起请求时，就需要各个模块之间进行协作，这样不可避免的要进行模块之间的调用。此时，我们的系统架构就会发生变化：

![](https://oscimg.oschina.net/oscnet/up-88b13599539e855487f181729dcdf460c13.png)

在这里，大家可以看到，**模块之间的调用，变得越来越复杂，而且模块之间还存在强耦合**。例如 A 调用 B ，那么就要在 A 中写上 B 的地址，也意味着 B 的部署位置要固定，同时，如果以后 B 要进行集群化部署， A 也需要修改，非常麻烦，此时就需要注册中心了。

## 2 Eureka 简介

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

## 3 搭建 Eureka 注册中心

### 3.1 单机模式

创建 Spring Boot 项目 `eureka-server` ，添加 `Eureka Server` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-fff939a1ae53b029a4d13e874c5bac3a539.png)

最终的依赖如下：

```xml
<dependencies>
      <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
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
 
项目创建成功后，在项目启动类上添加 `@EnableEurekaServer` 注解，标记该项目是一个 Eureka Server ，如下：

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }

}
```

接下来，在 `application.properties` 配置文件中添加基本配置信息，如下：

```properties
# 给当前服务取一个名字
spring.application.name=eureka
# 设置端口号
server.port=1111

# Eureka Server 也是一个普通的微服务，所以当它还是一个注册中心的时候，他会有两层身份：1.注册中心；2.普通服务。
# 默认情况下，会把自己注册到自己上面来，设置为 false 时，表示当前项目不要注册到注册中心上
eureka.client.register-with-eureka=false
# 表示是否从 Eureka Server 上获取注册信息
eureka.client.fetch-registry=false
```

---

配置完成后，就可以启动项目了。如果在项目启动时，遇到 `java.lang.TypeNotPresentException: Type javax.xml.bind.JAXBContext not present` 异常，这是因为 JDK9 以上，移除了 JAXB ，只需要我们手动引入 JAXB 即可。

```xml 
<dependency>
  <groupId>javax.xml.bind</groupId>
  <artifactId>jaxb-api</artifactId>
  <version>2.3.0</version>
</dependency>
<dependency>
  <groupId>com.sun.xml.bind</groupId>
  <artifactId>jaxb-impl</artifactId>
  <version>2.3.0</version>
</dependency>
<dependency>
  <groupId>org.glassfish.jaxb</groupId>
  <artifactId>jaxb-runtime</artifactId>
  <version>2.3.0</version>
</dependency>
<dependency>
  <groupId>javax.activation</groupId>
  <artifactId>activation</artifactId>
  <version>1.1.1</version>
</dependency>
```

项目启动成功后，访问 [http://127.0.0.1:1111](http://127.0.0.1:1111) 就可以查看 Eureka 后台管理页面了：

![](https://oscimg.oschina.net/oscnet/up-0591458c91349885a527134398570f1a7e2.png)

### 3.2 集群模式

使用了注册中心之后，**所有的服务都要通过服务注册中心来进行信息交换**。服务注册中心的稳定性就非常重要了，一旦服务注册中心掉线，会影响到整个系统的稳定性。所以，在实际开发中，服务注册中心一般都是以集群的形式出现的。**Eureka 集群，实际上就是启动多个 Eureka 实例，多个 Eureka 实例之间，互相注册，互相同步数据，共同组成一个 Eureka 集群**。

搭建 Eureka 集群，首先我们需要一点准备工作，修改电脑的 `hosts` 文件（ C:\Windows\System32\drivers\etc\hosts ）：

`127.0.0.1 eureka-a eureka-b`

---

创建 Spring Boot 项目 `eureka-servercluster` ，添加 `Eureka Server` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-fff939a1ae53b029a4d13e874c5bac3a539.png)

最终的依赖如下：

```xml
<dependencies>
      <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
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
 
项目创建成功后，在项目启动类上添加 `@EnableEurekaServer` 注解，标记该项目是一个 Eureka Server ，如下：

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerclusterApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerclusterApplication.class, args);
    }

}
```

在 `src/main/resources` 目录下，新增两个配置文件，分别如下：

`application-a.properties`：

```properties
# 给当前服务取一个名字
spring.application.name=eureka
# 设置端口号
server.port=1111

eureka.instance.hostname=eureka-a
# Eureka Server 也是一个普通的微服务，所以当它还是一个注册中心的时候，他会有两层身份：1.注册中心；2.普通服务。
# 默认情况下，会把自己注册到自己上面来，设置为 false 时，表示当前项目不要注册到注册中心上
eureka.client.register-with-eureka=true
# 表示是否从 Eureka Server 上获取注册信息
eureka.client.fetch-registry=true
# a 服务要注册到 b 上面
eureka.client.service-url.defaultZone=http://eureka-b:1112/eureka
```

`application-b.properties`：

```properties
# 给当前服务取一个名字
spring.application.name=eureka
# 设置端口号
server.port=1112

eureka.instance.hostname=eureka-b
# Eureka Server 也是一个普通的微服务，所以当它还是一个注册中心的时候，他会有两层身份：1.注册中心；2.普通服务。
# 默认情况下，会把自己注册到自己上面来，设置为 false 时，表示当前项目不要注册到注册中心上
eureka.client.register-with-eureka=true
# 表示是否从 Eureka Server 上获取注册信息
eureka.client.fetch-registry=true
# b 服务要注册到 a 上面
eureka.client.service-url.defaultZone=http://eureka-a:1111/eureka
```

---

配置完成后，对当前项目打包，在命令行启动两个 Eureka 实例。两个启动命令分别如下：

```bash
java -jar eureka-servercluster-0.0.1-SNAPSHOT.jar --spring.profiles.active=a
java -jar eureka-servercluster-0.0.1-SNAPSHOT.jar --spring.profiles.active=b
```

项目启动成功后，就可以查看 Eureka 后台管理页面了，两个服务之间互相注册，共同给组成一个集群。

eureka-a：[http://127.0.0.1:1111](http://127.0.0.1:1111)

![](https://oscimg.oschina.net/oscnet/up-2759daf7e25f9cf5409eb7728d2c02299bc.png)

eureka-b：[http://127.0.0.1:1112](http://127.0.0.1:1112)

![](https://oscimg.oschina.net/oscnet/up-5e617a97dc512195558f5ba382951d8efe7.png)

## 4 搭建 Eureka 客户端

### 4.1 服务注册

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
 
### 4.2 服务消费

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

#### 4.2.1 服务调用 - HttpURLConnection

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

#### 4.2.2 服务调用 - RestTemplate（推荐）

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


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.top](https://cxy35.top)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)