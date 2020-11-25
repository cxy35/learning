---
title: Spring Boot 整合 Redis
date: 2019-12-15 15:20:38
categories: Spring Boot
tags: [Spring Boot, Redis]
toc: true
---
学习在 Spring Boot 中使用 Redis 来实现数据存储。在 Spring Boot 中，默认集成的 Redis 就是 **`Spring Data Redis`** ，默认底层的连接池使用了 `lettuce` ，可以自行修改为自己的熟悉的，例如 `Jedis` 。
<!-- more -->

## 1 创建工程并配置

创建 Spring Boot 项目 `spring-boot-redis` ，添加 `Web/Redis` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-3d36fe2c124205997f83040f9bd4eb02d95.png)

之后手动在 pom 文件中添加 `commos-pool2` 依赖，最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-pool2</artifactId>
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

接着在 `application.properties` 配置文件中添加 Redis 相关信息的配置和 Redis 连接池的配置，如下：

```properties
# Redis 配置
spring.redis.host=192.168.71.62
spring.redis.port=6379
spring.redis.database=0
spring.redis.password=000000

# 连接池配置， Spring Boot 默认用的是 lettuce ，而不是 Jedis ，需增加 commons-pool2 依赖
spring.redis.lettuce.pool.min-idle=5
spring.redis.lettuce.pool.max-idle=10
spring.redis.lettuce.pool.max-active=8
spring.redis.lettuce.pool.max-wait=1ms
spring.redis.lettuce.shutdown-timeout=100ms
```

## 2 使用

配置完成之后，就可以直接在 Service 或 Controller 中注入 `StringRedisTemplate` 或者 `RedisTemplate` 来使用。新建 `HelloController` ，如下：

```java
@RestController
public class HelloController {
    // 模板1：<Object, Object>
    @Autowired
    RedisTemplate redisTemplate;
    // 模板2：<String, String>
    @Autowired
    StringRedisTemplate stringRedisTemplate;

    @GetMapping("/set")
    public void set() {
        // RedisTemplate 中，key 默认的序列化方案是 JdkSerializationRedisSerializer， key 不易读
        // 下面这句可修改为 StringRedisSerializer
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        ValueOperations ops = redisTemplate.opsForValue();
        ops.set("k1", "v1");
    }

    @GetMapping("/get")
    public void get(){
        ValueOperations ops = redisTemplate.opsForValue();
        System.out.println(ops.get("k1"));
    }

    @GetMapping("/set2")
    public void set2() {
        // StringRedisTemplate 中，key 默认的序列化方案是 StringRedisSerializer
        ValueOperations<String, String> ops = stringRedisTemplate.opsForValue();
        ops.set("k2", "v2");
    }

    @GetMapping("/get2")
    public void get2() {
        ValueOperations<String, String> ops = stringRedisTemplate.opsForValue();
        System.out.println(ops.get("k2"));
    }
}
```

## 3 源码解读

Redis 对应的自动化配置类是 `org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration` ，源码如下：

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnClass({RedisOperations.class})
@EnableConfigurationProperties({RedisProperties.class})
@Import({LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class})
public class RedisAutoConfiguration {
    public RedisAutoConfiguration() {
    }

    @Bean
    @ConditionalOnMissingBean(
        name = {"redisTemplate"}
    )
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        RedisTemplate<Object, Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    @Bean
    @ConditionalOnMissingBean
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
}
```

自动化配置类说明：

- 该配置类在 `RedisOperations` 存在的情况下才会生效（即项目中引入了 Spring Data Redis ）。
- 通过 `@EnableConfigurationProperties` 导入 `application.properties` 中配置的属性。
- 然后导入连接池信息（如果存在的话）。
- 最后提供了两个默认 Bean : `RedisTemplate` 和 `StringRedisTemplate` ，其中 StringRedisTemplate 是 RedisTemplate 的子类，两个的方法基本一致，不同之处主要体现在操作的数据类型不同。 RedisTemplate 中的两个泛型都是 Object ，意味者存储的 key 和 value 都可以是一个对象，而 StringRedisTemplate 的两个泛型都是 String ，意味者 StringRedisTemplate 的 key 和 value 都只能是字符串。如果开发者没有提供相关的 Bean ，这两个配置就会生效，否则不会生效。

**注意：自动化配置，只能配置单机的 Redis ，但如果是 Redis 集群，则所有的东西都需要我们自己来配置**。

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-nosql/spring-boot-redis](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-nosql/spring-boot-redis)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)