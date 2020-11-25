---
title: Spring Boot 整合 Spring Cache + Ehcache（实现数据缓存）
date: 2019-12-28 11:35:31
categories: Spring Boot
tags: [Spring Boot, Spring Cache, Ehcache]
toc: true
---
学习在 Spring Boot 中整合 `Spring Cache + Ehcache` ，实现数据的缓存。 Spring Cache 统一了缓存江湖的门面，它提供统一的接口，实现可以是 `Redis` 或 `Ehcache` 或其他支持这种规范的缓存框架，他们的关系类似于 JDBC 与各种数据库驱动，本文使用 `Ehcache` 实现。 Ehcache 也是 Java 领域比较优秀的缓存方案之一，但在 Redis 一统江湖的时代， Ehcache 渐渐有点没落了。
<!-- more -->

## 1 创建工程

创建 Spring Boot 项目 `spring-boot-springcache-ehcache` ，添加 `Web/Spring Cache` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-457e35a40096eb879cafe71152a8d4d499e.png)

之后手动在 pom 文件中添加 `ehcache` 依赖，最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>net.sf.ehcache</groupId>
        <artifactId>ehcache</artifactId>
        <version>2.10.6</version>
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

## 2 Ehcache 配置

在 `resources` 目录下，新增配置文件 `ehcache.xml` ，如下：

```xml
<ehcache>
    <!--
       name：缓存名称。
       maxElementsInMemory：缓存最大个数。
       eternal：对象是否永久有效，一但设置了，timeout将不起作用。
       timeToIdleSeconds：设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
       timeToLiveSeconds：设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。
       overflowToDisk：当内存中对象数量达到maxElementsInMemory时，Ehcache将会对象写到磁盘中。
       diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
       maxElementsOnDisk：硬盘最大缓存个数。
       diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false.
       diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。
       memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。
       clearOnFlush：内存数量最大时是否清除。
       diskStore：则表示临时缓存的硬盘目录。
    -->
    <diskStore path="java.io.tmpdir/ehcache"/>
    <defaultCache
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            overflowToDisk="false"
            diskPersistent="false"
            diskExpiryThreadIntervalSeconds="120"
    />
    <cache name="cache-user"
           maxElementsInMemory="10000"
           eternal="false"
           timeToIdleSeconds="120"
           overflowToDisk="true"
           diskPersistent="true"
           diskExpiryThreadIntervalSeconds="600"/>
</ehcache>
```

配置说明见上文注释。

Ehcache 配置文件的默认位置在 `classpath` 下，默认名称为 `ehcache.xml` 。支持个性化，在 `application.properties` 配置文件中配置，如下：

```properties
# 自定义 Ehcache 配置文件的位置和名称
# spring.cache.ehcache.config=classpath:myEhcache.xml
```

## 3 使用

首先在项目启动类上增加 `@EnableCaching` 注解，开始缓存，如下：

```java
@SpringBootApplication
@EnableCaching // 开启缓存
public class SpringBootSpringcacheEhcacheApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootSpringcacheEhcacheApplication.class, args);
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
// 指定缓存名称
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
class SpringBootSpringcacheEhcacheApplicationTests {

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

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-cache/spring-boot-springcache-ehcache](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-cache/spring-boot-springcache-ehcache)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)