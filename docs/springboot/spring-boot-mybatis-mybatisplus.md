学习在 Spring Boot 中整合 MyBatis-Plus。MyBatis-Plus（简称 MP）是一个 MyBatis 的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。
<!-- more -->

## 1 概述

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）
- **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- **分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库
- **内置性能分析插件**：可输出 Sql 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作

## 2 基本使用

1. 创建项目，引入依赖

创建 Spring Boot 项目 `spring-boot-mybatis-mybatisplus` ，添加 `Web/MySQL Driver/Lombok` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-52b7eab0299fc0404733e20eb445492deb3.png)

之后手动在 pom 文件中添加 `Druid/MyBatis-Plus` 依赖（Spring Boot 版本），最终的依赖如下：

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
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.4.2</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

接着在 `application.yml` 配置文件中添加数据库相关信息的配置，如下：

```yml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/cxy35?useUnicode=true&characterEncoding=utf-8&autoReconnect=true&autoReconnectForPools=true&serverTimezone=Asia/Shanghai
    username: root
    password: '000000'

mybatis-plus:
#  mapper-locations: classpath:/mapper/**/*.xml # 指定 mapper.xml 路径
  global-config:
    db-config:
      id-type: auto # 全局默认主键类型设置为自增
      # table-prefix: t_ # 全局配置表名前缀，这样 MP 拼接 SQL 时会自动添加该前缀
      # logic-delete-field: deleted # 全局逻辑删除的实体字段名
      # logic-delete-value: 1 # 逻辑已删除值(默认为1)
      # logic-not-delete-value: 0 # 逻辑未删除值(默认为0)
  configuration:
    auto-mapping-behavior: partial # 只对非嵌套的 resultMap 进行自动映射
    map-underscore-to-camel-case: true # 开启自动驼峰命名规则映射

# [MyBatis-Plus 代码生成器](https://github.com/cxy35/generators/tree/master/generator-mybatisplus)
```

---

2. 新建实体类

手动新建或用代码生成器生成 `User` 实体类，并增加相关注解，如下：

```java
@Data
@TableName("t_user")
public class User implements Serializable {
    @TableId(value = "id", type = IdType.AUTO)
    private Integer id;
    // 以实体对象作为条件查询时，配置该字段使用 like 进行拼接
    @TableField(condition = SqlCondition.LIKE)
    private String username;
    private String password;
    private Boolean enabled;
    private Boolean locked;
    private String address;
    private String nickName;
    private Date createTime;
    private Date updateTime;
}
```

注解说明：

- `@TableName`：注解在类上，指定类和数据库表的映射关系。**实体类的类名（转成小写后）和数据库表名相同时**，可以不指定该注解。
- `@TableId`：注解在实体类的某一字段上，**表示这个字段对应数据库表的主键**。当主键名为 id 时（表中列名为 id，实体类中字段名为 id），无需使用该注解显式指定主键，mp 会自动关联。若类的字段名和表的列名不一致，可用 `value` 属性指定表的列名。另，这个注解有个重要的属性 `type`，用于指定主键策略：
    - `IdType.AUTO`：数据库 ID 自增，**依赖于数据库**。在插入操作生成 SQL 语句时，不会插入主键这一列。
    - `IdType.INPUT`：需要手动设置主键。完全依赖于用户输入。实体对象中主键ID是什么，插入到数据库时就设置什么，若有值便设置值，若为 null 则设置 null。oracle 的序列主键需要使用这种方式。
    - `IdType.NONE`：在实体对象中主键 ID 为空时，才会自动生成，使用**主键的全局策略**（默认的主键全局策略是基于雪花算法的自增 ID，可在 `application.yml` 配置文件中修改，如：`mybatis-plus.global-config.db-config.id-type: auto`）。
    - `IdType.ASSIGN_ID`：在实体对象中主键 ID 为空时，才会自动生成，使用**雪花算法**。
    - `IdType.ASSIGN_UUID`：在实体对象中主键 ID 为空时，才会自动生成，使用 **UUID**。
- `@TableField`：注解在某一字段上，指定 Java 实体类的字段和数据库表的列的映射关系。这个注解有如下几个应用场景：
    - **排除非表字段**：若Java实体类中某个字段，不对应表中的任何列，它只是用于保存一些额外的，或组装后的数据，则可以设置 `exist` 属性为 `false`，这样在对实体对象进行插入时，会忽略这个字段。排除非表字段也可以通过其他方式完成，如使用 `static` 或 `transient` 关键字，但个人觉得不是很合理，不做赘述。
    - **字段验证策略**：通过 `insertStrategy`，`updateStrategy`，`whereStrategy` 属性进行配置，可以控制在实体对象进行插入，更新，或作为WHERE条件时，对象中的字段要如何组装到SQL语句中。
    - **字段填充策略**：通过 `fill` 属性指定，字段为空时会进行自动填充，如在插入时自动填充：`@TableField(fill = FieldFill.INSERT)`，需要额外实现自动填充处理器 `MetaObjectHandler`。
- `@Version`：乐观锁。
- `@EnumValue`：注解在枚举字段上。
- `@TableLogic`：**逻辑删除**，在实体类的对应字段上使用，进行单独配置。另外，也可通过 `application.yml` 配置文件做全局配置。
- `@KeySequence`：序列主键策略（oracle）。
- `@InterceptorIgnore`：插件过滤规则。

---

3. 新建 Mapper 接口

```java
public interface UserMapper extends BaseMapper<User> {
    // 非必须，自定义 SQL（注解方式），支持多表联查
    // SQL 中不写 WHERE 关键字，且固定使用 ${ew.customSqlSegment}
    @Select("select * from user ${ew.customSqlSegment}")
    List<User> selectListAll(@Param(Constants.WRAPPER) Wrapper<User> wrapper);

    // 非必须，自定义 SQL（XML 方式），支持多表联查
    List<User> selectListAll2(Wrapper<User> wrapper);
}
```

这里继承 Mybatis-Plus 中的 BaseMapper 接口，这样就自动拥有了其中的所有接口，不用写 XXXMapper.xml 文件，相关接口如下：

![](https://oscimg.oschina.net/oscnet/up-0adc8dc3abe34b04e4720ab3fca7a1b3372.png)

- `insert(T entity)`：插入一条记录。
- `deleteById(Serializable id)`：根据主键 id 进行删除。
- `deleteBatchIds`：根据主键 id 进行批量删除。
- `deleteByMap`：根据 Map 进行删除（Map 中的 key 为列名，value 为值，根据列和值进行**等值匹配**）。
- `delete(Wrapper<T> wrapper)`：根据条件构造器 wrapper 进行删除。
- `update(T entity, Wrapper<T> wrapper)`：根据实体 entity 和条件构造器 wrapper 进行更新。
- `updateById(T entity)`：根据入参 entity 的 id（主键）进行更新，对于 **entity 中非空的属性**，会出现在 UPDATE 语句的 SET 后面，即 entity 中非空的属性，会被更新到数据库。
- `selectById(Serializable id)`：根据主键 id 进行查找。
- `selectBatchIds(Collection idList)`：根据主键 id 进行批量查找。
- `selectByMap(Map<String,Object> map)`：根据 map 中指定的列名和列值进行**等值匹配**查找。
- `selectList(Wrapper<T> wrapper)`：根据条件构造器 wrapper 进行查询。
- `selectMaps(Wrapper<T> wrapper)`：根据 wrapper 条件，查询记录，将查询结果封装为一个 Map，Map 的 key 为结果的列，value 为值。
- `selectObjs(Wrapper<T> wrapper)`：只会返回第一个字段（第一列）的值，其他字段会被舍弃。
- `selectCount(Wrapper<T> wrapper)`：查询满足条件的总数，注意，使用这个方法，不能调用 QueryWrapper 的 select 方法设置要查询的列了，这个方法会自动添加 `select count(1)`。
- `...`

---

4. 配置 Mapper 接口的扫描和分页插件

新建 Mybatis-Plus 配置类，如下：。

```java
@Configuration
@MapperScan("com.cxy35.sample.springboot.mybatis.mybatisplus.mapper")
public class MyBatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();

        // 配置分页查询
        PaginationInnerInterceptor paginationInnerInterceptor = new PaginationInnerInterceptor();
        paginationInnerInterceptor.setDbType(DbType.MYSQL);
        // 设置请求的页面大于最大页后操作，true调回到首页，false 继续请求，默认false
        // paginationInnerInterceptor.setOverflow(false);
        // 设置最大单页限制数量，默认 500 条，-1 不受限制
        // paginationInnerInterceptor.setMaxLimit(500L);
        // 开启 count 的 join 优化，只针对部分 left join
        // paginationInnerInterceptor.setOptimizeJoin(true);
        interceptor.addInnerInterceptor(paginationInnerInterceptor);

        // 配置动态表名
        /*DynamicTableNameInnerInterceptor dynamicTableNameInnerInterceptor = new DynamicTableNameInnerInterceptor();
        HashMap<String, TableNameHandler> map = new HashMap<>();
        // 对于 t_user 表，进行动态表名设置（t_user_1,t_user_2）
        map.put("t_user", (sql, tableName) -> {
            String _ = "_";
            int random = new Random().nextInt(2) + 1;
            return tableName + _ + random; // 若返回null, 则不会进行动态表名替换, 还是会使用 t_user
        });
        dynamicTableNameInnerInterceptor.setTableNameHandlerMap(map);
        interceptor.addInnerInterceptor(dynamicTableNameInnerInterceptor);*/

        return interceptor;
    }
}
```

---

5. 测试 Mapper

在测试类中注入 `UserMapper` 完成测试，如下：

```java
@SpringBootTest
class UserMapperTests {

    @Autowired
    UserMapper userMapper;

    @Test
    public void insert() {
        User user = new User();
        user.setUsername("zhangsan");
        user.setPassword("123456");
        user.setAddress("杭州");
        user.setNickName("zs");
        user.setCreateTime(new Date());
        user.setUpdateTime(new Date());
        userMapper.insert(user);
    }

    @Test
    public void deleteById() {
        userMapper.deleteById(5);
    }

    @Test
    public void updateById() {
        User user = new User();
        user.setId(5);
        user.setUsername("zhangsan2");
        user.setPassword("654321");
        user.setNickName("zs2");
        user.setAddress("上海");
        user.setUpdateTime(new Date());
        userMapper.updateById(user);
    }

    @Test
    public void update() {
        // 普通更新，按对象
        User user = new User();
        user.setAddress("上海");
        user.setUpdateTime(new Date());
        LambdaUpdateWrapper<User> wrapper = new LambdaUpdateWrapper<>();
        wrapper.likeRight(User::getUsername, "zhang");
        userMapper.update(user, wrapper);

        // 普通更新，按字段
        LambdaUpdateWrapper<User> wrapper2 = new LambdaUpdateWrapper<>();
        wrapper2.likeRight(User::getUsername, "zhang").set(User::getAddress, "上海").set(User::getUpdateTime, new Date());
        userMapper.update(null, wrapper);

        // 链式调用更新，按字段
        LambdaUpdateChainWrapper<User> wrapper3 = new LambdaUpdateChainWrapper<>(userMapper);
        wrapper3.likeRight(User::getUsername, "zhang").set(User::getAddress, "上海").set(User::getUpdateTime, new Date()).update();
    }

    @Test
    public void selectById() {
        User user = userMapper.selectById(5);
        System.out.println(user);
    }

    @Test
    public void selectByMap() {
        // 根据 map 中指定的列名和列值进行等值匹配查找
        Map<String, Object> columnMap = new HashMap<>();
        columnMap.put("address", "杭州");
        List<User> users = userMapper.selectByMap(columnMap);
        users.forEach(System.out::println);
    }

    @Test
    public void selectList() {
        // 查询所有
        List<User> allUser = userMapper.selectList(null);
        allUser.forEach(System.out::println);

        // 字段作为条件
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.likeRight("username", "zhang");
        List<User> users = userMapper.selectList(wrapper);
        users.forEach(System.out::println);

        // 实体对象作为条件（默认会以实体对象中的非空属性，构建等值匹配的 WHERE 条件，可通过在实体类的字段上配置 @TableField 注解中的 condition 属性进行改变）
        User user = new User();
        user.setUsername("zhang");
        QueryWrapper<User> wrapper2 = new QueryWrapper<>(user);
        List<User> users2 = userMapper.selectList(wrapper2);
        users2.forEach(System.out::println);

        // 链式调用查询
        LambdaQueryChainWrapper<User> wrapper3 = new LambdaQueryChainWrapper<>(userMapper);
        List<User> users3 = wrapper3.likeRight(User::getUsername, "zhang").list();
        users3.forEach(System.out::println);
    }

    @Test
    public void selectPage() {
        Page<User> page = new Page<>(1, 3);
        // Page<User> page = new Page<>(1, 3, false); // 不查总记录数
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.between("id", 1, 20).eq("enabled", 1).orderByDesc("id");
        // wrapper.lambda().between(User::getId, 1, 20).eq(User::getEnabled, 1).orderByDesc(User::getId);
        // LambdaQueryWrapper<User> wrapper2 = new LambdaQueryWrapper<>();
        // LambdaQueryWrapper<User> wrapper2 = Wrappers.<User>lambdaQuery();
        // wrapper2.between(User::getId, 1, 20).eq(User::getEnabled, 1).orderByDesc(User::getId);
        Page<User> pageResult = userMapper.selectPage(page, wrapper);
        System.out.println("总记录数：" + pageResult.getTotal());
        System.out.println("总页数：" + pageResult.getPages());
        System.out.println("当前页：" + pageResult.getCurrent());
        System.out.println("每页记录数：" + pageResult.getSize());
        System.out.println("是否存在上一页：" + pageResult.hasPrevious());
        System.out.println("是否存在下一页：" + pageResult.hasNext());
        // 获取分页查询结果
        List<User> records = pageResult.getRecords();
        records.forEach(System.out::println);
    }

    @Test
    // 根据 wrapper 条件，查询记录，将查询结果封装为一个 Map，Map 的 key 为结果的列，value 为值
    public void selectMaps() {
        // 使用场景：只查部分列
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.select("id", "username", "address").likeRight("username", "zhang");
        List<Map<String, Object>> users = userMapper.selectMaps(wrapper);
        users.forEach(System.out::println);

        // 使用场景：进行数据统计（表中需增加 age 字段）
        QueryWrapper<User> wrapper2 = new QueryWrapper<>();
        wrapper2.select("address", "avg(age) avg_age", "min(age) min_age", "max(age) max_age")
                .groupBy("address").having("sum(age) < {0}", 500);
        List<Map<String, Object>> users2 = userMapper.selectMaps(wrapper);
        users2.forEach(System.out::println);
    }

    @Test
    // 只会返回第一个字段（第一列）的值，其他字段会被舍弃
    public void selectObjs() {
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.select("id", "username").likeRight("username", "zhang");
        List<Object> users = userMapper.selectObjs(wrapper);
        // 得到的结果，只封装了第一列的id
        users.forEach(System.out::println);
    }

    @Test
    // 查询满足条件的总数，注意，使用这个方法，不能调用 QueryWrapper 的 select 方法设置要查询的列了，这个方法会自动添加 `select count(1)`
    public void selectCount() {
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.likeRight("username", "zhang");
        Integer count = userMapper.selectCount(wrapper);
        System.out.println(count);
    }

    @Test
    // 条件构造器的诸多方法中，均可以指定一个 boolean 类型的参数 condition，用来决定该条件是否加入最后生成的WHERE语句中
    public void testCondition() {
        String username = "zhang";
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.likeRight(username != null && username != "", "username", username);
        Integer count = userMapper.selectCount(wrapper);
        System.out.println(count);
    }

    @Test
    // allEq方法传入一个map，用来做等值匹配
    public void testAllEq() {
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        Map<String, Object> param = new HashMap<>();
        param.put("address", "杭州");
        param.put("username", "zhangsan");
        param.put("enabled", null);
        wrapper.allEq(param, false); // 忽略 map 中 value 为 null 的元素
        wrapper.allEq((k, v) -> !"address".equals(k), param); // 过滤掉 map 中 key 为 name 的元素
        List<User> users = userMapper.selectList(wrapper);
        users.forEach(System.out::println);
    }

    @Test
    // lambda 条件构造器，支持 lambda 表达式，可以不必像普通条件构造器一样，以字符串形式指定列名，它可以直接以实体类的方法引用来指定列，比较优雅
    public void testLambda() {
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
        // LambdaQueryWrapper<User> wrapper = Wrappers.<User>lambdaQuery();
        // LambdaQueryWrapper<User> wrapper = new QueryWrapper<User>().lambda();
        wrapper.eq(User::getAddress, "杭州").likeRight(User::getUsername, "zhang");
        List<User> users = userMapper.selectList(wrapper);
        users.forEach(System.out::println);
    }

    @Test
    // 链式 lambda 条件构造器，代码写起来非常简洁
    public void testChain() {
        // 查询
        LambdaQueryChainWrapper<User> wrapper = new LambdaQueryChainWrapper<>(userMapper);
        List<User> users2 = wrapper.eq(User::getAddress, "杭州").likeRight(User::getUsername, "zhang").list();
        users2.forEach(System.out::println);

        // 更新
        LambdaUpdateChainWrapper<User> wrapper2 = new LambdaUpdateChainWrapper<>(userMapper);
        wrapper2.likeRight(User::getUsername, "zhang").set(User::getAddress, "上海").set(User::getUpdateTime, new Date()).update();

        // 删除
        LambdaUpdateChainWrapper<User> wrapper3 = new LambdaUpdateChainWrapper<>(userMapper);
        wrapper3.eq(User::getEnabled, false).remove();
    }
}
```

---

6. 新建 Service 接口和实现类

```java
public interface UserService extends IService<User> {
}
```

```java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
}
```

这里继承 Mybatis-Plus 中的 IService 接口，这样就自动拥有了其中的所有接口，相关接口如下：

![](https://oscimg.oschina.net/oscnet/up-1fba84e2aa41cda5e52ae7b7898579f690b.png)

---

7. 测试 Service

在测试类中注入 `UserService` 完成测试，如下：

```java
@SpringBootTest
class UserServiceTests {

    @Autowired
    UserService userService;

    @Test
    public void save() {
        User user = new User();
        user.setUsername("zhangsan");
        user.setPassword("123456");
        user.setAddress("杭州");
        user.setNickName("zs");
        user.setCreateTime(new Date());
        user.setUpdateTime(new Date());
        userService.save(user);
    }

    @Test
    public void removeById() {
        userService.removeById(6);
    }

    @Test
    public void updateById() {
        User user = new User();
        user.setId(6);
        user.setUsername("zhangsan2");
        user.setPassword("654321");
        user.setNickName("zs2");
        user.setAddress("上海");
        user.setUpdateTime(new Date());
        userService.updateById(user);
    }

    @Test
    public void update() {
        // 普通更新，按对象
        User user = new User();
        user.setAddress("上海");
        user.setUpdateTime(new Date());
        LambdaUpdateWrapper<User> wrapper = new LambdaUpdateWrapper<>();
        wrapper.likeRight(User::getUsername, "zhang");
        userService.update(user, wrapper);

        // 普通更新，按字段
        LambdaUpdateWrapper<User> wrapper2 = new LambdaUpdateWrapper<>();
        wrapper2.likeRight(User::getUsername, "zhang").set(User::getAddress, "上海").set(User::getUpdateTime, new Date());
        userService.update(null, wrapper);

        // 链式调用更新，按字段
        userService.lambdaUpdate().likeRight(User::getUsername, "zhang").set(User::getAddress, "上海").set(User::getUpdateTime, new Date()).update();
    }

    @Test
    public void getById() {
        User user = userService.getById(6);
        System.out.println(user);
    }

    @Test
    public void getOne() {
        LambdaQueryWrapper<User> wrapper = Wrappers.<User>lambdaQuery();
        wrapper.likeRight(User::getUsername, "zhang");
        // 第二参数指定为false,使得在查到了多行记录时,不抛出异常,而返回第一条记录
        User user = userService.getOne(wrapper, false);
        System.out.println(user);
    }

    @Test
    public void listByMap() {
        // 根据 map 中指定的列名和列值进行等值匹配查找
        Map<String, Object> columnMap = new HashMap<>();
        columnMap.put("address", "杭州");
        List<User> users = userService.listByMap(columnMap);
        users.forEach(System.out::println);
    }

    @Test
    public void list() {
        // 查询所有
        List<User> allUser = userService.list();
        allUser.forEach(System.out::println);

        // 字段作为条件
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.likeRight("username", "zhang");
        List<User> users = userService.list(wrapper);
        users.forEach(System.out::println);

        // 实体对象作为条件（默认会以实体对象中的非空属性，构建等值匹配的 WHERE 条件，可通过在实体类的字段上配置 @TableField 注解中的 condition 属性进行改变）
        User user = new User();
        user.setUsername("zhang");
        QueryWrapper<User> wrapper2 = new QueryWrapper<>(user);
        List<User> users2 = userService.list(wrapper2);
        users2.forEach(System.out::println);

        // 链式调用查询
        List<User> users3 = userService.lambdaQuery().likeRight(User::getUsername, "zhang").list();
        users3.forEach(System.out::println);
    }

    @Test
    public void page() {
        Page<User> page = new Page<>(1, 3);
        // Page<User> page = new Page<>(1, 3, false); // 不查总记录数
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.between("id", 1, 20).eq("enabled", 1).orderByDesc("id");
        // wrapper.lambda().between(User::getId, 1, 20).eq(User::getEnabled, 1).orderByDesc(User::getId);
        // LambdaQueryWrapper<User> wrapper2 = new LambdaQueryWrapper<>();
        // LambdaQueryWrapper<User> wrapper2 = Wrappers.<User>lambdaQuery();
        // wrapper2.between(User::getId, 1, 20).eq(User::getEnabled, 1).orderByDesc(User::getId);
        Page<User> pageResult = userService.page(page, wrapper);
        System.out.println("总记录数：" + pageResult.getTotal());
        System.out.println("总页数：" + pageResult.getPages());
        System.out.println("当前页：" + pageResult.getCurrent());
        System.out.println("每页记录数：" + pageResult.getSize());
        System.out.println("是否存在上一页：" + pageResult.hasPrevious());
        System.out.println("是否存在下一页：" + pageResult.hasNext());
        // 获取分页查询结果
        List<User> records = pageResult.getRecords();
        records.forEach(System.out::println);
    }

    @Test
    // lambda 条件构造器，支持 lambda 表达式，可以不必像普通条件构造器一样，以字符串形式指定列名，它可以直接以实体类的方法引用来指定列，比较优雅
    public void testLambda() {
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
        // LambdaQueryWrapper<User> wrapper = Wrappers.<User>lambdaQuery();
        // LambdaQueryWrapper<User> wrapper = new QueryWrapper<User>().lambda();
        wrapper.eq(User::getAddress, "杭州").likeRight(User::getUsername, "zhang");
        List<User> users = userService.list(wrapper);
        users.forEach(System.out::println);
    }

    @Test
    // 链式 lambda 条件构造器，代码写起来非常简洁
    public void testChain() {
        // 查询
        List<User> users = userService.lambdaQuery().eq(User::getAddress, "杭州").likeRight(User::getUsername, "zhang").list();
        users.forEach(System.out::println);

        // 更新
        userService.lambdaUpdate().likeRight(User::getUsername, "zhang").set(User::getAddress, "上海").set(User::getUpdateTime, new Date()).update();

        // 删除
        userService.lambdaUpdate().eq(User::getEnabled, false).remove();
    }
}
```

## 3 条件构造器

mp 让我觉得极其方便的一点在于其提供了强大的条件构造器 `Wrapper`，可以非常方便的构造 WHERE 条件。条件构造器主要涉及到3个类：父类 `AbstractWrapper`，子类 `QueryWrapper` 和 `UpdateWrapper`。在 `AbstractWrapper` 中提供了非常多的方法用于构建 WHERE 条件，而 `QueryWrapper` 针对 `SELECT` 语句，提供了 `select()` 方法，可自定义需要查询的列，而 `UpdateWrapper` 针对 `UPDATE` 语句，提供了 `set()` 方法，用于构造 `set` 语句。条件构造器也支持 `lambda` 表达式，写起来非常舒爽。

下面对 `AbstractWrapper` 中用于构建 SQL 语句中的 WHERE 条件的方法进行部分列举：

- `eq`：equals，等于
- `allEq`：all equals，全等于
- `ne`：not equals，不等于
- `gt`：greater than ，大于 `>`
- `ge`：greater than or equals，大于等于 `≥`
- `lt`：less than，小于 `<`
- `le`：less than or equals，小于等于 `≤`
- `between`：相当于 SQL 中的 `BETWEEN`
- `notBetween`
- `like`：模糊匹配。`like("name", "黄")`，相当于 SQL 的 `name like '%黄%'`
- `likeRight`：模糊匹配右半边。`likeRight("name", "黄")`，相当于 SQL 的 `name like '黄%'`
- `likeLeft`：模糊匹配左半边。`likeLeft("name", "黄")`，相当于 SQL 的 `name like '%黄'`
- `notLike`：`notLike("name", "黄")`，相当于 SQL 的 `name not like '%黄%'`
- `isNull`
- `isNotNull`
- `in`
- `and`：SQL 连接符 AND
- `or`：SQL 连接符 OR
- `apply`：用于拼接 SQL，该方法可用于数据库函数，并可以动态传参
- .......

**使用示例：**

```java
// 案例先展示需要完成的SQL语句，后展示Wrapper的写法  
  
// 1. 名字中包含佳，且年龄小于25  
// SELECT * FROM user WHERE name like '%佳%' AND age < 25  
QueryWrapper<User> wrapper = new QueryWrapper<>();  
wrapper.like("name", "佳").lt("age", 25);  
List<User> users = userMapper.selectList(wrapper);  
// 下面展示SQL时，仅展示WHERE条件；展示代码时, 仅展示Wrapper构建部分  
  
// 2. 姓名为黄姓，且年龄大于等于20，小于等于40，且email字段不为空  
// name like '黄%' AND age BETWEEN 20 AND 40 AND email is not null  
wrapper.likeRight("name","黄").between("age", 20, 40).isNotNull("email");  
  
// 3. 姓名为黄姓，或者年龄大于等于40，按照年龄降序排列，年龄相同则按照id升序排列  
// name like '黄%' OR age >= 40 order by age desc, id asc  
wrapper.likeRight("name","黄").or().ge("age",40).orderByDesc("age").orderByAsc("id");  
  
// 4.创建日期为2021年3月22日，并且直属上级的名字为李姓  
// date_format(create_time,'%Y-%m-%d') = '2021-03-22' AND manager_id IN (SELECT id FROM user WHERE name like '李%')  
wrapper.apply("date_format(create_time, '%Y-%m-%d') = {0}", "2021-03-22") // 建议采用{index}这种方式动态传参, 可防止SQL注入  
    .inSql("manager_id", "SELECT id FROM user WHERE name like '李%'");  
// 上面的apply, 也可以直接使用下面这种方式做字符串拼接，但当这个日期是一个外部参数时，这种方式有SQL注入的风险  
wrapper.apply("date_format(create_time, '%Y-%m-%d') = '2021-03-22'");  
  
// 5. 名字为王姓，并且（年龄小于40，或者邮箱不为空）  
// name like '王%' AND (age < 40 OR email is not null)  
wrapper.likeRight("name", "王").and(q -> q.lt("age", 40).or().isNotNull("email"));  
  
// 6. 名字为王姓，或者（年龄小于40并且年龄大于20并且邮箱不为空）  
// name like '王%' OR (age < 40 AND age > 20 AND email is not null)  
wrapper.likeRight("name", "王").or(  
    q -> q.lt("age",40)  
      .gt("age",20)  
      .isNotNull("email")  
  );  
  
// 7. (年龄小于40或者邮箱不为空) 并且名字为王姓  
// (age < 40 OR email is not null) AND name like '王%'  
wrapper.nested(q -> q.lt("age", 40).or().isNotNull("email"))  
    .likeRight("name", "王");  
  
// 8. 年龄为30，31，34，35  
// age IN (30,31,34,35)  
wrapper.in("age", Arrays.asList(30,31,34,35));  
// 或  
wrapper.inSql("age","30,31,34,35");  
  
// 9. 年龄为30，31，34，35, 返回满足条件的第一条记录  
// age IN (30,31,34,35) LIMIT 1  
wrapper.in("age", Arrays.asList(30,31,34,35)).last("LIMIT 1");  
  
// 10. 只选出id, name 列 (QueryWrapper 特有)  
// SELECT id, name FROM user;  
wrapper.select("id", "name");  
  
// 11. 选出id, name, age, email, 等同于排除 manager_id 和 create_time  
// 当列特别多, 而只需要排除个别列时, 采用上面的方式可能需要写很多个列, 可以采用重载的select方法，指定需要排除的列  
wrapper.select(User.class, info -> {  
   String columnName = info.getColumn();  
   return !"create_time".equals(columnName) && !"manager_id".equals(columnName);  
  });
```

## 4 划重点

- 条件构造器 `AbstractWrapper` 中提供了多个方法用于构造 SQL 语句中的 WHERE 条件，而其子类 `QueryWrapper` 额外提供了 `select` 方法，可以只选取特定的列，子类 `UpdateWrapper` 额外提供了 `set` 方法，用于设置 SQL 中的 `SET` 语句。除了普通的 `Wrapper`，还有基于 lambda 表达式的 `Wrapper`，如 `LambdaQueryWrapper`，`LambdaUpdateWrapper`，它们在构造 WHERE 条件时，直接以**方法引用**来指定 WHERE 条件中的列，比普通 Wrapper 通过字符串来指定要更加优雅。另，还有**链式 Wrapper**，如 `LambdaQueryChainWrapper`，它封装了 `BaseMapper`，可以更方便地获取结果。
- 条件构造器采用**链式调用**来拼接多个条件，条件之间默认以 `AND` 连接
- 当 `AND` 或 `OR` 后面的条件需要被括号包裹时，将括号中的条件以 `lambda` 表达式形式，作为参数传入 `and()` 或 `or()`。特别的，当 `()` 需要放在 WHERE 语句的最开头时，可以使用 `nested()` 方法
- 条件表达式时当需要传入自定义的 SQL 语句，或者需要调用数据库函数时，可用 `apply()` 方法进行 SQL 拼接
- 条件构造器中的各个方法可以通过一个 `boolean` 类型的变量 `condition`，来根据需要灵活拼接 WHERE 条件（仅当 `condition` 为 `true` 时会拼接 SQL 语句）
- 使用 `lambda` 条件构造器，可以通过 `lambda` 表达式，直接使用实体类中的属性进行条件构造，比普通的条件构造器更加优雅
- 若 mp 提供的方法不够用，可以通过**自定义 SQL**（原生 mybatis）的形式进行扩展开发
- 使用 mp 进行分页查询时，需要创建一个分页拦截器（`Interceptor`），注册到 Spring 容器中，随后查询时，通过传入一个分页对象（Page 对象）进行查询即可。单表查询时，可以使用 `BaseMapper` 提供的 `selectPage` 或 `selectMapsPage` 方法。复杂场景下（如多表联查），使用自定义 SQL。
- AR 模式可以直接通过操作实体类来操作数据库。让实体类继承自 Model 即可。

## 5 代码生成器

- [MyBatis-Plus 代码生成器](https://github.com/cxy35/generators/tree/master/generator-mybatisplus)

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-dao/spring-boot-mybatis-mybatisplus](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-dao/spring-boot-mybatis-mybatisplus)
- [MyBatis-Plus，看这一篇就够了！](https://mp.weixin.qq.com/s/W_Q1dnXuopCb9Gm322NPlg)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.top](https://cxy35.top)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)