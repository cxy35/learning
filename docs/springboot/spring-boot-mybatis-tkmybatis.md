---
title: Spring Boot 整合 MyBatis 通用 Mapper（TkMybatis）
date: 2020-05-20 20:27:46
categories: Spring Boot
tags: [Spring Boot, TkMybatis]
toc: true
---
学习在 Spring Boot 中整合 MyBatis 通用 Mapper（TkMybatis）。通用 Mapper 是一个可以实现任意 MyBatis 通用方法的框架，项目提供了常规的增删改查操作以及 Example 相关的单表操作。通用 Mapper 是为了解决 MyBatis 使用中 90% 的基本操作，使用它可以很方便的进行开发，可以节省开发人员大量的时间。
<!-- more -->

## 1 概述

通用 Mapper 都可以极大的方便开发人员。可以随意的按照自己的需要选择通用方法，还可以很方便的开发自己的通用方法。

极其方便的使用 MyBatis 单表的增删改查。

支持单表操作，不支持通用的多表联合查询。

使用通用 Mapper 可以无 xml 文件实现数据库操作，只需要继承 TkMybatis 中相关的 Mapper 接口即可。但如果有特殊需求，可以自定义 XXXMapper.xml 文件，实现复杂 sql 语句的操作。

**为什么需要通用 Mapper ？**

我个人最早用 MyBatis 时，先是完全手写，然后用上了 **MyBatis 代码生成器（简称为 MBG）**，在使用 MBG 过程中，发现一个很麻烦的问题，如果数据库字段变化很频繁，就需要反复重新生成代码，并且由于 MBG 覆盖生成代码和追加方式生成 XML，导致每次重新生成都需要大量的比对修改。除了这个问题外，还有一个问题，仅仅基础的增删改查等方法，就已经产生了大量的 XML 内容，还没有添加一个自己手写的方法，代码可能就已经几百行了，内容多，看着比较碍事。

因为很多人都在使用 MBG，MBG 中定义了很多常用的单表方法，为了解决前面提到的问题，也为了兼容 MBG 的方法避免项目重构太多，在 MBG 的基础上结合了部分 JPA 注解产生了通用 Mapper。通用 Mapper 可以很简单的让你获取基础的单表方法，也很方便扩展通用方法。使用通用 Mapper 可以极大的提高你的工作效率。

> 说明：我们也可以改造 MBG ，比如自动生成一套基本的 model/mapper/service 等，与表对应，不去做修改，自定义的都写在对应的另一个子类上，这样，当表字段修改后，只需全部重新生成上述基本的那些文件，再手动修改自定义的文件（如果有需要）即可。

## 2 基本使用

1. 创建项目，引入依赖

创建 Spring Boot 项目 `spring-boot-mybatis-tkmybatis` ，添加 `Web/MySQL Driver` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-ed5e587fbe983dde365ca23568525309de6.png)

之后手动在 pom 文件中添加 `Druid/Mapper` 依赖（Spring Boot 版本），最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.1.10</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
        <version>5.1.27</version>
    </dependency>
    <dependency>
        <groupId>tk.mybatis</groupId>
        <artifactId>mapper-spring-boot-starter</artifactId>
        <version>2.1.5</version>
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

接着在 `application.properties` 配置文件中添加数据库相关信息的配置，如下：

```properties
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.username=root
spring.datasource.password=000000
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/cxy35?useUnicode=true&characterEncoding=utf-8&autoReconnect=true&autoReconnectForPools=true

logging.level.com.cxy35.sample.springboot.mybatis.tkmybatis.mapper=debug

# [Mybatis 通用 Mapper 代码生成器](https://github.com/cxy35/generators/tree/master/generator-mapper)
```

---

2. 新建实体类

手动新建或用代码生成器生成 `User` 实体类，并增加相关注解，如下：

```java
@Table(name = "t_user")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY, generator = "JDBC")
    private Integer id;
    private String username;
    private String password;
    private Boolean enabled;
    private Boolean locked;
    // @Column(name = "c_address")
    private String address;
    private String nickName;
    private Date createTime;
    private Date updateTime;

    // getter/setter
}
```

注解说明：

- `@Table`：指定该实体类对应的表名，如果表名为 t_user ，类名为 TUser ，则可以不需要此注解（默认驼峰命名规则）。
- `@Column`：指定该属性对应的列名，如果列名为 create_time ，属性名为 createTime ，则可以不需要此注解（默认驼峰命名规则）。
- `@Id`：标识该字段对应数据库表的主键 id 。   
- `@GeneratedValue`：指定主键 id 生成规则，其中 strategy 表示使用数据库自带的主键生成策略， generator 配置为"JDBC"，在数据插入完毕之后，会自动将主键id填充到实体类中，类似普通 mapper.xml 中配置的 selectKey 标签。

---

3. 新建 Mapper 接口

```java
public interface UserMapper extends Mapper<User> {
}
```

这里继承 TkMybatis 中最基本的一个通用 Mapper 接口，这样就自动拥有了这个 Mapper 中的一些接口，不用写 XXXMapper.xml 文件。

相关接口如下：

![](https://oscimg.oschina.net/oscnet/up-b98d6d08583a85b31839410657168a8b376.png)

---

除了 Mapper 接口，官方还提供了一些几个好用的通用 Mapper 接口，都可以用来继承使用，汇总如下：

- Mapper 接口：

![](https://oscimg.oschina.net/oscnet/up-fa4e9a0170bc36e17089fd8b29d4be599cd.png)

IdsMapper 接口：

![](https://oscimg.oschina.net/oscnet/up-428cbaede325eb4bde5dbcc6f61966a946f.png)

ConditionMapper 接口：

![](https://oscimg.oschina.net/oscnet/up-8490a3ffe0722f4ac51537b15b964af7368.png)

- MySqlMapper 接口：

![](https://oscimg.oschina.net/oscnet/up-7b9fdb051e51f5444b22ff67e9e7fd6f05b.png)

- SqlServerMapper 接口：

![](https://oscimg.oschina.net/oscnet/up-ad5697a880e120685970584116dda416338.png)

> 当然，我们也可以根据自己的实际业务需求，抽取通用业务逻辑，自定义通用 Mapper 接口，参考：[通用 Mapper 进阶实例：为什么好久都没更新了？](https://blog.csdn.net/isea533/category_9262342.html) 。

4. 配置 Mapper 接口的扫描

可以在启动类上或自定义 MyBatis 的配置类上，通过 `@MapperScan` 注解配置。

```java
@SpringBootApplication
@tk.mybatis.spring.annotation.MapperScan(basePackages = "com.cxy35.sample.springboot.mybatis.tkmybatis.mapper")
public class SpringBootMybatisTkmybatisApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootMybatisTkmybatisApplication.class, args);
    }

}
```

---

5. 测试

在测试类中注入 `UserMapper` 完成测试，如下：

```java
@SpringBootTest
class SpringBootMybatisTkmybatisApplicationTests {

    @Autowired
    UserMapper userMapper;

    @Test
    public void insertSelective() {
        User user = new User();
        user.setUsername("zhangsan");
        user.setPassword("123456");
        user.setAddress("杭州");
        user.setNickName("zs");
        user.setCreateTime(new Date());
        user.setUpdateTime(new Date());
        userMapper.insertSelective(user);
    }

    @Test
    public void deleteByPrimaryKey() {
        userMapper.deleteByPrimaryKey(2);
    }

    @Test
    public void updateByPrimaryKeySelective() {
        User user = new User();
        user.setId(2);
        user.setUsername("zhangsan2");
        user.setPassword("654321");
        user.setNickName("zs2");
        user.setAddress("上海");
        userMapper.updateByPrimaryKeySelective(user);
    }

    @Test
    public void selectByPrimaryKey() {
        User user = userMapper.selectByPrimaryKey(4);
        System.out.println(user);
    }

    @Test
    public void selectAll() {
        List<User> users = userMapper.selectAll();
        System.out.println(users);
    }

    @Test
    public void selectByExample() {
        Example example = new Example(User.class);
        Example.Criteria criteria = example.createCriteria();
        criteria.andLike("username", "zhangsan%");
        criteria.andEqualTo("address", "杭州");
        List<User> users = userMapper.selectByExample(example);
        System.out.println(users);
    }
}
```

## 3 自定义 XXXMapper.xml（非必须）

虽然大多数复杂的需求，都能通过 TkMyBatis 的组合完成操作。但如果有特殊需求，可以自定义 XXXMapper.xml 文件，实现复杂 sql 语句的操作，这里以联表查询为例。

在 `UserMapper.java` 所在包下新建 `UserMapper.xml` ，增加如下内容：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- 测试用，非必须 -->
<mapper namespace="com.cxy35.sample.springboot.mybatis.tkmybatis.mapper.UserMapper">
    <select id="selectByRoleId" parameterType="java.lang.Integer" resultType="com.cxy35.sample.springboot.mybatis.tkmybatis.pojo.User">
        SELECT
            u.*
        FROM
            t_user u
        INNER JOIN t_user_role ur ON u.id = ur.user_id
        WHERE
            ur.role_id = #{roleId, jdbcType=INTEGER};
    </select>
</mapper>
```

上述 xml 文件与普通 xml 文件的不同之处在于：这里不需要使用 resultMap 进行字段的映射。当然，如果想在返回的 Map 中新增返回字段映射，直接添加新的字段即可。

**注意：不要在 xml 文件中写 TkMyBatis 中已经有的一些基础方法，否则会报错，提示方法重复。**

接着，修改 UserMapper.java ，增加对应的接口：

```java
public interface UserMapper extends Mapper<User> {
    // 测试用，非必须
    List<User> selectByRoleId(Integer roleId);
}
```

---

上述 UserMapper.xml 放在 UserMapper.java 所在的包下面，会被自动扫描到。但在项目打包时会被忽略掉，因此需要在 pom.xml 中配置 Maven 构建时的资源路径。

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.xml</include>
            </includes>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
        </resource>
    </resources>
</build>
```

关于 XXXMapper.xml 文件位置，还有其他方案，这里不再赘述，具体可参考 [Spring Boot 整合 MyBatis](https://mp.weixin.qq.com/s/zvOBkU-BKAk-4yhwboZbzA) 。

最后在测试类中增加方法，完成测试：

```java
@Test
public void selectByRoleId() {
    List<User> users = userMapper.selectByRoleId(2);
    System.out.println(users);
}
```

## 4 代码生成器

- [Mybatis 通用 Mapper 代码生成器](https://github.com/cxy35/generators/tree/master/generator-mapper)

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-dao/spring-boot-mybatis-tkmybatis](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-dao/spring-boot-mybatis-tkmybatis)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)