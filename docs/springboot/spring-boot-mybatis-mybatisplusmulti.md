学习在 Spring Boot 中使用 MyBatis-Plus 多数据源来操作不同的数据库，使用 `dynamic-datasource-spring-boot-starter` 框架实现，他是一个基于 Spring Boot 的快速集成多数据源的启动器。本文基于 [Spring Boot 整合 MyBatis-Plus](docs/springboot/spring-boot-mybatis-mybatisplus.md) 一文修改，包括代码实现，请先阅读。
<!-- more -->

## 1 特性

- 支持**数据源分组**，适用于多种场景：纯粹多库、读写分离、一主多从、混合模式。
- 支持数据库敏感配置信息**加密**ENC()。
- 支持每个数据库独立初始化表结构 schema 和数据库 database。
- 支持**自定义注解**，需继承 DS(3.2.0+)。
- 提供对 Druid，Mybatis-Plus，P6sy，Jndi 的快速集成。
- 简化Druid和HikariCp配置，提供**全局参数配置**。配置一次，全局通用。
- 提供**自定义数据源来源**方案。
- 提供项目启动后**动态增加移除数据源**方案。
- 提供 Mybatis 环境下的**纯读写分离**方案。
- 提供使用**spel 动态参数**解析数据源方案。内置 spel，session，header，支持自定义。
- 支持**多层数据源嵌套切换**。（ServiceA >>> ServiceB >>> ServiceC）。
- 提供对 shiro，sharding-jdbc，quartz 等第三方库集成的方案,注意事项和示例。
- 提供**基于seata的分布式事务方案**。 附：不支持原生 spring 事务。
- 提供**本地多数据源事务方案**。 附：不支持原生 spring 事务。

## 2 约定

- 本框架只做**切换数据源**这件核心的事情，并**不限制你的具体操作**，切换了数据源可以做任何 CRUD。
- 配置文件所有以下划线 `_` 分割的数据源**首部**即为组的名称，相同组名称的数据源会放在一个组下。
- 切换数据源可以是组名，也可以是具体数据源名称。组名则切换时采用负载均衡算法切换。
- 默认的数据源名称为 `master` ，你可以通过 `spring.datasource.dynamic.primary` 修改。
- 方法上的注解优先于类上注解。
- 强烈建议只在 service 的类和方法上添加注解，不建议在 mapper 上添加注解。

## 3 使用方法

1. 增加 `dynamic-datasource-spring-boot-starter` 依赖（注意版本需要与 MyBatis-Plus 匹配），如下：

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
    <version>3.3.0</version>
</dependency>
<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-all</artifactId>
    <version>1.4.1</version>
</dependency>
```

2. 在 `application.yml` 配置文件中添加数据库相关信息的配置，如下：

```yml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    dynamic:
      primary: master #设置默认的数据源或者数据源组,默认值即为master
      strict: false # 设置严格模式，默认 false 不启动. 启动后在未匹配到指定数据源时候会抛出异常，不启动则使用默认数据源.
      datasource:
        master:
          driver-class-name: com.mysql.jdbc.Driver # 3.2.0 开始支持 SPI 可省略此配置
          url: jdbc:mysql://127.0.0.1:3306/cxy35?useUnicode=true&characterEncoding=utf-8&autoReconnect=true&autoReconnectForPools=true&serverTimezone=Asia/Shanghai
          username: root
          password: '000000'
        slave_1:
          driver-class-name: com.mysql.jdbc.Driver # 3.2.0 开始支持 SPI 可省略此配置
          url: jdbc:mysql://127.0.0.1:3306/cxy35_2?useUnicode=true&characterEncoding=utf-8&autoReconnect=true&autoReconnectForPools=true&serverTimezone=Asia/Shanghai
          username: root
          password: '000000'
        # slave_2:
          # url: ENC(xxxxx) # 内置加密，使用请查看详细文档
          # username: ENC(xxxxx)
          # password: ENC(xxxxx)
          # driver-class-name: com.mysql.jdbc.Driver
          # schema: db/schema.sql # 配置则生效，自动初始化表结构
          # data: db/data.sql # 配置则生效，自动初始化数据
          # continue-on-error: true # 默认 true，初始化失败是否继续
          # separator: ";" # sql 默认分号分隔符
        # ......省略
        # 以上会配置一个默认库 master，一个组 slave 下有两个子库 slave_1，slave_2

mybatis-plus:
  #  mapper-locations: classpath:/mapper/**/*.xml # 指定 mapper.xml 路径
  global-config:
    db-config:
      id-type: auto # 全局默认主键类型设置为自增
      # logic-delete-field: deleted # 全局逻辑删除的实体字段名
      # logic-delete-value: 1 # 逻辑已删除值(默认为1)
      # logic-not-delete-value: 0 # 逻辑未删除值(默认为0)
  configuration:
    auto-mapping-behavior: partial # 只对非嵌套的 resultMap 进行自动映射
    map-underscore-to-camel-case: true # 开启自动驼峰命名规则映射

# [MyBatis-Plus 代码生成器](https://github.com/cxy35/generators/tree/master/generator-mybatisplus)
```

```
# 多主多从                      纯粹多库（记得设置primary）                   混合配置
spring:                               spring:                               spring:
  datasource:                           datasource:                           datasource:
    dynamic:                              dynamic:                              dynamic:
      datasource:                           datasource:                           datasource:
        master_1:                             mysql:                                master:
        master_2:                             oracle:                               slave_1:
        slave_1:                              sqlserver:                            slave_2:
        slave_2:                              postgresql:                           oracle_1:
        slave_3:                              h2:                                   oracle_2:
```

3. 使用 `@DS` 切换数据源。

`@DS` 可以注解在方法上或类上，同时存在就近原则，方法上注解**优先于**类上注解。

|注解|结果|
|:-|:-|
|没有 @DS|默认数据源|
|@DS("dsName")|dsName 可以为组名也可以为具体某个库的名称|

---

修改 `UserServiceImpl`，在类上增加 `@DS("master")`，默认走主库，除非在方法在添加 `@DS("slave")` 才走从库。

```java
@Service
@DS("master")
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
    @Autowired
    UserMapper userMapper;

    /**
     * 重写父类的方法或自定义方法，指定数据源，会覆盖类上的数据源
     *
     * @return
     */
    @DS("slave")
    @Override
    public List<User> list() {
        return userMapper.selectList(null);
    }
}
```

4. 测试：

```java
@SpringBootTest
class UserServiceTests {

    @Autowired
    UserService userService;

    @Test
    public void save() {
        User user = new User();
        user.setUsername("test2");
        user.setPassword("123456");
        user.setAddress("宁波");
        user.setNickName("测试2");
        user.setCreateTime(new Date());
        user.setUpdateTime(new Date());
        userService.save(user);
    }

    @Test
    public void list() {
        List<User> users = userService.list();
        System.out.println(users);
    }
}
```

## 4 注意事项

如果集成了 `Druid`，会出现如下错误：

```
***************************
APPLICATION FAILED TO START
***************************

Description:

Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.

Reason: Failed to determine a suitable driver class

......

Caused by: org.springframework.boot.autoconfigure.jdbc.DataSourceProperties$DataSourceBeanCreationException: Failed to determine a suitable driver class
```

怎么解决？在启动类上排除掉 `Druid` 对应的自动配置类即可解决，如下：

```java
@SpringBootApplication(exclude = DruidDataSourceAutoConfigure.class)
public class SpringBootMybatisMybatisplusmultiApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootMybatisMybatisplusmultiApplication.class, args);
    }

}
```

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-dao/spring-boot-mybatis-mybatisplusmulti](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-dao/spring-boot-mybatis-mybatisplusmulti)
- MyBatis-Plus 多数据源文档：[https://mybatis.plus/guide/dynamic-datasource.html](https://mybatis.plus/guide/dynamic-datasource.html)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)