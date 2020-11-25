---
title: Spring Boot 自定义 Starter
date: 2019-11-11 16:17:11
categories: Spring Boot
tags: [Spring Boot, Starter]
toc: true
---
认识 Spring Boot 中的自动化配置，并手把手带你写一个自己的 Starter 。
<!-- more -->

## 1 认识 Starter

Spring Boot 中的 Starter 为我们完成了很多自动化配置，使得我们可以很轻松的搭建一个生产级的开发环境。其实 Starter 并不难，都是 Spring + Spring MVC 中的基础知识点实现的，他的核心就是条件注解 `@Conditional` ，当 classpath 下存在某个 Class 时，某个配置才会生效。

除了 Spring Boot 官方提供的 Starter 之外，第三方公司的功能针对 Spring Boot 一般都会有 1 个 Starter 提供自动化配置类，命名都有一定的规范，有兴趣的可以看下源码，比如：
- Thymeleaf ： `spring-boot-autoconfigure` jar 包中的
`org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration` 类，对应的源码解读可查看文章 [Spring Boot 整合 Thymeleaf](https://mp.weixin.qq.com/s/3E27wfdlEQVjJb1hZ5Rz9g) 。
- Redis ： `spring-boot-autoconfigure` jar 包中的 `org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration` 类。
- Mybatis ： `mybatis-spring-boot-starter` jar 包中的 `org.mybatis.spring.boot.autoconfigure.MybatisAutoConfiguration` 类。

## 2 自定义 Starter

首先创建一个普通的 Maven 项目 spring-boot-mystarter ，创建完成后，添加官方的 Starter 依赖 spring-boot-autoconfigure ，如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-autoconfigure</artifactId>
        <version>2.2.0.RELEASE</version>
    </dependency>
</dependencies>
```

在 src/main/java 下相应的包中新建 HelloService 类，如下：

```java
public class HelloService {
    private String name;
    private String msg;

    public String sayHello() {
        return name + " say " + msg + " !";
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}
```

在 src/main/java 下相应的包中新建 HelloProperties 类，用来接收 application.properties 中注入的值，如下：

```java
// 类型安全的属性注入，指定配置的前缀
@ConfigurationProperties(prefix = "cxy35")
public class HelloProperties {
    private static final String DEFAULT_NAME = "默认名称";
    private static final String DEFAULT_MSG = "默认消息";
    private String name = DEFAULT_NAME;
    private String msg = DEFAULT_MSG;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}
```

在 src/main/java 下相应的包中新建 HelloServiceAutoConfiguration 类，就是我们的自动化配置类，如下：

```java
@Configuration
@EnableConfigurationProperties(HelloProperties.class)
@ConditionalOnClass(HelloService.class)
public class HelloServiceAutoConfiguration {
    @Autowired
    HelloProperties helloProperties;

    @Bean
    HelloService helloService() {
        HelloService helloService = new HelloService();
        helloService.setName(helloProperties.getName());
        helloService.setMsg(helloProperties.getMsg());
        return helloService;
    }
}
```

自动化配置类说明：

- `@Configuration` 注解表示这是一个配置类。
- `@EnableConfigurationProperties` 注解表示开启 ConfigurationProperties ，即使得我们上面 HelloProperties 类上配置的 @ConfigurationProperties 生效。
- `@ConditionalOnClass` 表示当项目 classpath 下存在 HelloService 时，当前的自动化配置类才会生效。
- 首先注入 HelloProperties ，这个实例中含有我们在 `application.properties` 中配置的相关数据。
- 最后提供一个 HelloService 的实例，将 HelloProperties 中的值注入进去。

接下来在 src/main/resources/META-INF 下中新建 spring.factories 文件，指我们的自动化配置类，如下：

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.cxy35.sample.springboot.mystarter.HelloServiceAutoConfiguration
```

这个文件干嘛用的呢？我们的 Spring Boot 项目的启动类都有一个 @SpringBootApplication 注解，这个注解的定义如下：

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM,
				classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
}
```

其中 @EnableAutoConfiguration 表示启用 Spring 应用程序上下文的自动化配置，该注解会自动导入一个名为 AutoConfigurationImportSelector 的类，而这个类会去读取一个名为 spring.factories 的文件, spring.factories 中则定义需要加载的自动化配置类，我们打开任意一个框架的 Starter ，都能看到它有一个 spring.factories 文件，例如 MyBatis 的 Starter 如下：

![](https://oscimg.oschina.net/oscnet/up-daafd4aa1d6acdbe1026104fc5c984ebf0b.png)

最后需要将这个自动化配置类安装到本地仓库，然后在其他项目中使用即可。安装方式很简单，在 IntelliJ IDEA 中，点击右边的 Maven ，然后选择 Lifecycle 中的 install ，双击即可，如下：

![](https://oscimg.oschina.net/oscnet/up-5276235f71777009278913b7026900dd089.png)

或者使用 Maven 命令安装也行。

## 3 使用自定义 Starter

首先创建一个普通的 Spring Boot 项目 spring-boot-usemystarter ，增加 Web 依赖，创建完成后，再手动添加我们自定义的 Starter 依赖 spring-boot-mystarter ，如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
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
    <dependency>
        <groupId>com.cxy35.sample</groupId>
        <artifactId>spring-boot-mystarter</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

加入上述依赖后，我们的项目中就有了一个默认的 HelloService 实例，可以通过在单元测试中注入该实例使用，如下：

```java
@SpringBootTest
public class SpringBootUsemystarterApplicationTests {

    @Autowired
    HelloService helloService;

    @Test
    void contextLoads() {
        System.out.println(helloService.sayHello());
    }
}
```

也可以在 `application.properties` 配置文件中添加我们自定义的配置，如下：

```properties
cxy35.name=自定义名称
cxy35.msg=自定义消息
```

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-starter](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-starter)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)