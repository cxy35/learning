---
title: Spring Boot 整合 JdbcTemplate
date: 2019-12-04 09:15:43
categories: Spring Boot
tags: [Spring Boot, JdbcTemplate]
toc: true
---
学习在 Spring Boot 中使用 JdbcTemplate 来操作数据库。 JdbcTemplate 是 Spring 自带的，虽然功能没有 MyBatis 强大，但配置和使用简单。
<!-- more -->

## 1 创建工程并配置

创建 Spring Boot 项目 `spring-boot-jdbctemplate` ，添加 `Web/JDBC/MySQL` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-fdcae0dc81e9b6c2b895d96a6ccd5477fc8.png)

之后手动在 pom 文件中添加 Druid 数据库连接池依赖（Spring Boot 版本），最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
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
spring.datasource.username=root
spring.datasource.password=000000
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/cxy35?useUnicode=true&characterEncoding=utf-8&autoReconnect=true&autoReconnectForPools=true
```

## 2 使用

新建 User 实体类，如下：

```java
public class User {
    private Integer id;
    private String username;
    private String address;

    // getter/setter
}
```

新建 UserService ，注入 `JdbcTemplate` ，实现增删改查的业务代码，如下：

```java
@Service
public class UserService {
    @Autowired
    JdbcTemplate jdbcTemplate;

    // 新增
    public int addUser(User user) {
        return jdbcTemplate.update("insert into t_user (username,address) values (?,?);", user.getUsername(), user.getAddress());
    }

    // 新增，主键回填
    public int addUser2(User user) {
        KeyHolder keyHolder = new GeneratedKeyHolder();
        int r = jdbcTemplate.update(new PreparedStatementCreator() {
            @Override
            public PreparedStatement createPreparedStatement(Connection connection) throws SQLException {
                PreparedStatement ps = connection.prepareStatement("insert into t_user (username,address) values (?,?);", Statement.RETURN_GENERATED_KEYS);
                ps.setString(1, user.getUsername());
                ps.setString(2, user.getAddress());
                return ps;
            }
        }, keyHolder);
        user.setId(keyHolder.getKey().intValue());
        return r;
    }

    // 删除
    public int deleteUserById(Integer id) {
        return jdbcTemplate.update("delete from t_user where id=?", id);
    }

    // 修改
    public int updateUserById(User user) {
        return jdbcTemplate.update("update t_user set username=?,address=? where id=?", user.getUsername(), user.getAddress(), user.getId());
    }

    // 查询，使用 RowMapper 手动实现数据库字段和对象属性的映射
    public List<User> getAllUsers() {
        return jdbcTemplate.query("select * from t_user", new RowMapper<User>() {
            @Override
            public User mapRow(ResultSet resultSet, int i) throws SQLException {
                User user = new User();
                int id = resultSet.getInt("id");
                String username = resultSet.getString("username");
                String address = resultSet.getString("address");
                user.setId(id);
                user.setUsername(username);
                user.setAddress(address);
                return user;
            }
        });
    }

    // 查询，使用 BeanPropertyRowMapper 简单实现，前提是数据库字段和对象属性名称一致
    public List<User> getAllUsers2() {
        return jdbcTemplate.query("select * from t_user", new BeanPropertyRowMapper<>(User.class));
    }
}
```

JdbcTemplate API 说明：

1. update ：实现新增/修改/删除操作。
2. query ：实现查询操作。

---

最后在测试类中注入 userService 完成测试，如下：

```java
@SpringBootTest
class SpringBootJdbctemplateApplicationTests {

    @Autowired
    UserService userService;

    @Test
    public void addUser() {
        User user = new User();
        user.setUsername("zhangsan");
        user.setAddress("杭州");
        userService.addUser(user);
        System.out.println(user);
    }

    @Test
    public void addUser2() {
        User user = new User();
        user.setUsername("lisi");
        user.setAddress("北京");
        userService.addUser2(user);
        System.out.println(user);
    }

    @Test
    public void deleteUserById() {
        userService.deleteUserById(2);
    }

    @Test
    public void updateUserById() {
        User user = new User();
        user.setId(1);
        user.setUsername("zhangsan2");
        user.setAddress("上海");
        userService.updateUserById(user);
    }

    @Test
    public void getAllUsers() {
        List<User> allUsers = userService.getAllUsers();
        System.out.println(allUsers);
    }

    @Test
    public void getAllUsers2() {
        List<User> allUsers = userService.getAllUsers2();
        System.out.println(allUsers);
    }
}
```

## 3 源码解读

JdbcTemplate 对应的自动化配置类是 `org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration` ，源码如下：

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnClass({DataSource.class, JdbcTemplate.class})
@ConditionalOnSingleCandidate(DataSource.class)
@AutoConfigureAfter({DataSourceAutoConfiguration.class})
@EnableConfigurationProperties({JdbcProperties.class})
@Import({JdbcTemplateConfiguration.class, NamedParameterJdbcTemplateConfiguration.class})
public class JdbcTemplateAutoConfiguration {
    public JdbcTemplateAutoConfiguration() {
    }
}
```

从上面的源码可以看到，当存在 `DataSource` 和 `JdbcTemplate` 类时， `JdbcTemplateAutoConfiguration` 的配置生效。另外引入了 `JdbcTemplateConfiguration` 和 `NamedParameterJdbcTemplateConfiguration` ，源码如下：

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnMissingBean({JdbcOperations.class})
class JdbcTemplateConfiguration {
    JdbcTemplateConfiguration() {
    }

    @Bean
    @Primary
    JdbcTemplate jdbcTemplate(DataSource dataSource, JdbcProperties properties) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        Template template = properties.getTemplate();
        jdbcTemplate.setFetchSize(template.getFetchSize());
        jdbcTemplate.setMaxRows(template.getMaxRows());
        if (template.getQueryTimeout() != null) {
            jdbcTemplate.setQueryTimeout((int)template.getQueryTimeout().getSeconds());
        }

        return jdbcTemplate;
    }
}
```

如果自己没有提供 `JdbcOperations` 类型的 Bean ，系统就提供一个默认的 `JdbcTemplate` （ JdbcOperations 接口的一个实现）。

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnSingleCandidate(JdbcTemplate.class)
@ConditionalOnMissingBean({NamedParameterJdbcOperations.class})
class NamedParameterJdbcTemplateConfiguration {
    NamedParameterJdbcTemplateConfiguration() {
    }

    @Bean
    @Primary
    NamedParameterJdbcTemplate namedParameterJdbcTemplate(JdbcTemplate jdbcTemplate) {
        return new NamedParameterJdbcTemplate(jdbcTemplate);
    }
}
```

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-dao/spring-boot-jdbctemplate](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-dao/spring-boot-jdbctemplate)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)