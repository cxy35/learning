---
title: Spring Boot 整合 Spring Cache + Redis（实现数据缓存）
date: 2019-12-27 10:12:24
categories: Spring Boot
tags: [Spring Boot, Spring Cache, Redis]
toc: true
---
学习在 Spring Boot 中整合 `Spring Cache + Redis` ，实现数据的缓存。 Spring Cache 统一了缓存江湖的门面，它提供统一的接口，实现可以是 `Redis` 或 `Ehcache` 或其他支持这种规范的缓存框架，他们的关系类似于 JDBC 与各种数据库驱动，本文使用 `Redis` 实现。这种方式相对于自己手动通过 RedisTemplate 往 Redis 中缓存数据（参考 [Spring Boot 整合 Redis](https://mp.weixin.qq.com/s/oXwCwO0Ng24xvYo3DcXjDA) ）来说比较简单。
<!-- more -->

## 1 创建工程并配置

创建 Spring Boot 项目 `spring-boot-springcache-redis` ，添加 `Web/Spring Cache/Redis` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-c4a7abde06af004cae2561da97394bfc586.png)

之后手动在 pom 文件中添加 `commos-pool2` 依赖，最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
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

接着在 `application.properties` 配置文件中添加 Redis 相关信息的配置和 Redis 连接池的配置，还有缓存名称的配置，后续会用到，如下：

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

# 缓存名称
spring.cache.cache-names=cache-user
```

## 2 使用

首先在项目启动类上增加 `@EnableCaching` 注解，开始缓存，如下：

```java
@SpringBootApplication
@EnableCaching // 开启缓存
public class SpringBootSpringcacheRedisApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootSpringcacheRedisApplication.class, args);
    }

}
```

接着新建 `User` 实体类，如下：

```java
public class User implements Serializable {
    private Integer id;
    private String username;
    private String address;

    // getter/setter
}
```

最后新增 `UserService` ，如下：

```java
@Service
// 指定缓存名称，对应配置文件中 spring.cache.cache-names=cache-user
@CacheConfig(cacheNames = "cache-user")
public class UserService {

    // 注意要在启动类上配置开启缓存 @EnableCaching

    // 默认缓存的 key 为所有参数的值（可通过 key 或 keyGenerator 修改），缓存的 value 为方法的返回值
    // cache-user::1
    // 查询
    @Cacheable
    // @Cacheable(key = "#id")
    // @Cacheable(key = "#root.methodName")
    // @Cacheable(keyGenerator = "myKeyGenerator")
    public User getUserById(Integer id) {
        System.out.println("getUserById >>> " + id);
        User user = new User();
        user.setId(id);
        return user;
    }

    // 删除
    @CacheEvict
    public void deleteUserById(Integer id) {
        System.out.println("deleteUserById >>> " + id);
    }

    // 更新
    @CachePut(key = "#user.id")
    public User updateUserById(User user) {
        return user;
    }
}
```

缓存注解说明：

1. `@CacheConfig`

这个注解在**类**上使用，用来指定该类中所有方法使用的全局缓存名称。

2. `@Cacheable`

这个注解一般在**查询方法**上使用，表示将一个方法的返回值缓存起来，默认缓存的 key 为所有参数的值（可通过 key 或 keyGenerator 修改），缓存的 value 为方法的返回值。

如果方法有多个参数时，默认就使用多个参数来做 key ，如果只需要其中某一个参数做 key ，则可以在 `@Cacheable` 注解中，通过 `key` 属性来指定，如 `@Cacheable(key = "#id")` 。如果对 key 有复杂的要求，可以自定义 `keyGenerator` 。 Spring Cache 默认中提供了 `root` 对象，可以在不定义 keyGenerator 的情况下实现一些复杂的效果，如下：

![](https://oscimg.oschina.net/oscnet/up-9dcc96fc1290c4065aa02ed93a51a1ade43.png)

也可以通过 keyGenerator 自定义 key ，如 `@Cacheable(keyGenerator = "myKeyGenerator")` 。需要新增 `MyKeyGenerator` 配置类，如下：

```java
@Component
public class MyKeyGenerator implements KeyGenerator {
    @Override
    public Object generate(Object o, Method method, Object... objects) {
        // 自定义缓存的 key
        return method.getName() + ":" + Arrays.toString(objects);
    }
}
```

3. `@CacheEvict`

这个注解一般在**删除方法**上使用，当数据库中的数据删除后，相关的缓存数据也要自动清除。当然也可以配置按照某种条件删除（ `condition` 属性）或者配置清除所有缓存（ `allEntries` 属性）。

4. `@CachePut`

这个注解一般在**更新方法**上使用，当数据库中的数据更新后，缓存中的数据也要跟着更新，使用该注解，可以将方法的返回值自动更新到已经存在的 key 上。

---

下面开始测试，在测试类中注入 `UserService` ，如下：

```java
@SpringBootTest
class SpringBootSpringcacheRedisApplicationTests {

    @Autowired
    UserService userService;

    @Test
    void contextLoads() {
        User u1 = userService.getUserById(1);

//        userService.deleteUserById(1);

//        User user = new User();
//        user.setId(1);
//        user.setUsername("zhangsan");
//        user.setAddress("hangzhou");
//        userService.updateUserById(user);

        User u2 = userService.getUserById(1);

        System.out.println(u1);
        System.out.println(u2);
    }
}
```

## 3 源码解读

Redis 缓存对应的自动化配置类是 `org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration` ，部分源码如下：

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnClass({RedisConnectionFactory.class})
@AutoConfigureAfter({RedisAutoConfiguration.class})
@ConditionalOnBean({RedisConnectionFactory.class})
@ConditionalOnMissingBean({CacheManager.class})
@Conditional({CacheCondition.class})
class RedisCacheConfiguration {
    RedisCacheConfiguration() {
    }

    @Bean
    RedisCacheManager cacheManager(CacheProperties cacheProperties, CacheManagerCustomizers cacheManagerCustomizers, ObjectProvider<org.springframework.data.redis.cache.RedisCacheConfiguration> redisCacheConfiguration, ObjectProvider<RedisCacheManagerBuilderCustomizer> redisCacheManagerBuilderCustomizers, RedisConnectionFactory redisConnectionFactory, ResourceLoader resourceLoader) {
        RedisCacheManagerBuilder builder = RedisCacheManager.builder(redisConnectionFactory).cacheDefaults(this.determineConfiguration(cacheProperties, redisCacheConfiguration, resourceLoader.getClassLoader()));
        List<String> cacheNames = cacheProperties.getCacheNames();
        if (!cacheNames.isEmpty()) {
            builder.initialCacheNames(new LinkedHashSet(cacheNames));
        }

        redisCacheManagerBuilderCustomizers.orderedStream().forEach((customizer) -> {
            customizer.customize(builder);
        });
        return (RedisCacheManager)cacheManagerCustomizers.customize(builder.build());
    }

    // ......
}
```

上述源码中提供了一个 `RedisCacheManager` 类型的 Bean ，它间接实现了 Spring Cache 的接口，有了它我们就可以直接使用 Spring 中的缓存注解和接口了，而缓存的数据则会被自动存储到 Redis 中。

**注意：在单机的 Redis 中，系统会自动提供这个 Bean ，但如果是 Redis 集群，则需要我们自己来提供**。

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-cache/spring-boot-springcache-redis](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-cache/spring-boot-springcache-redis)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)