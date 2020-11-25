---
title: Spring Boot 整合 Jpa
date: 2019-12-10 10:46:31
categories: Spring Boot
tags: [Spring Boot, Jpa]
toc: true
---
学习在 Spring Boot 中使用 Jpa 来操作数据库。在 Spring Boot 中，使用的 Jpa 实际上是 **`Spring Data Jpa`** ， Spring Data 是 Spring 家族的一个子项目，用于简化 SQL 和 NoSQL 的访问，在 Spring Data 中，只要你的**方法名称符合规范**，它就知道你想干什么，不需要自己再去写 SQL 。
<!-- more -->

## 1 Jpa 简介

Jpa(Java Persistence API) ， Java 持久化 API ，它是一套 ORM 规范，而不是具体的实现。 Jpa 类似于 JDBC ，只提供规范，所有的数据库厂商提供实现（即具体的数据库驱动），在 Java 领域，大家熟知的 ORM(Object Relational Mapping) 框架可能主要是 Hibernate ，实际上，除了 Hibernate 之外，还有很多其他的 ORM 框架，例如：

- Batoo JPA
- DataNucleus (formerly JPOX)
- EclipseLink (formerly Oracle TopLink)
- IBM, for WebSphere Application Server
- JBoss with Hibernate
- Kundera
- ObjectDB
- OpenJPA
- OrientDB from Orient Technologies
- Versant Corporation JPA (not relational, object database)

Hibernate 只是 ORM 框架的一种，上面列出来的 ORM 框架都是支持 JPA 2.0 规范的。既然它是一个规范，不是具体的实现，那么必然就不能直接使用（类似于 JDBC 不能直接使用，必须要加了驱动才能用），我们使用的是具体的实现，在这里我们采用的实现实际上还是 Hibernate 。

## 2 创建工程并配置

创建 Spring Boot 项目 `spring-boot-jpa` ，添加 `Web/JPA/MySQL` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-d56d0986b2b4f777fece8ba276bc5013d93.png)

之后手动在 pom 文件中添加 Druid 数据库连接池依赖（Spring Boot 版本），最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
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

接着在 `application.properties` 配置文件中添加数据库相关信息的配置和 Jpa 的基本配置，如下：

```properties
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.username=root
spring.datasource.password=000000
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/cxy35?useUnicode=true&characterEncoding=utf-8&autoReconnect=true&autoReconnectForPools=true

# JPA配置
# 数据库为 MySQL
spring.jpa.database=mysql
# 在控制台打印 SQL
spring.jpa.show-sql=true
# 数据库平台
spring.jpa.database-platform=mysql
# 每次启动项目时，数据库初始化策略
spring.jpa.hibernate.ddl-auto=update
# 指定默认的存储引擎为 InnoDB ，否则默认情况下，自动创建表的时候会使用 MyISAM 作为表的存储引擎
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL57Dialect
```

## 3 使用

使用 ORM 框架（对象关系映射）我们不必再去创建表，框架会自动根据当前项目中的实体类创建相应的数据表。因此，这里首先创建一个 `Book` 实体类，如下：

```java
@Entity(name = "t_book")
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    // @Column(name = "c_name")
    private String name;
    private String author;

    // getter/setter
}
```

实体类说明：

- `@Entity` 注解: 表示这是一个实体类，在项目启动时会自动针对该类生成一张表，默认的表名为类名，可使用 name 属性自定义生成的表名。
- `@Id` 注解: 表示这个字段是一个 id 。
- `@GeneratedValue` 注解: 表示主键的自增长策略。
- 对于类中的其他属性，默认都会根据属性名在表中生成相应的字段，字段名和属性名相同。如果想要对字段进行定制，可以使用`@Column` 注解，去配置字段的名称，长度，是否为空等。

这时启动 Spring Boot 项目，会发现数据库中多了一个名为 `t_book` 的表。

---

下面开始定义接口操作数据库，新增 `BookDao` ，定义相关接口和 SQL ，如下：

```java
public interface BookDao extends JpaRepository<Book, Integer> {
    // 方法定义规范：
    // 1.按照 Spring Data 的规范，查询方法以 find/get/read 开头
    // 2.涉及条件查询时，条件的属性用条件关键字连接，要注意的是：条件属性以首字母大写
    // 3.支持属性的级联查询. 若当前类有符合条件的属性, 则优先使用, 而不使用级联属性. 若需要使用级联属性, 则属性之间使用 _ 进行连接

    // Book findById(Integer id);// 父类中定义了该方法

    Book findBookById(Integer id);

    Book findByIdAndName(Integer id, String name);

    List<Book> findByIdGreaterThan(Integer id);

    List<Book> findByIdLessThanOrNameContaining(Integer id, String name);

    Book getById(Integer id);

    Book getBookById(Integer id);

    Book readById(Integer id);

    Book readBookById(Integer id);

    // @Query 注解： JPQL 或 SQL
    @Query(value = "select b.* from t_book b where b.id>?1 and b.name=?2", nativeQuery = true)
    // 参数要按顺序
    List<Book> getByParam(Integer id, String name);

    @Query(value = "select b.* from t_book b where id>:id and name=:name", nativeQuery = true)
    // 使用 @Param ，参数可以不按顺序（推荐）
    List<Book> getByParam2(@Param("name") String name, @Param("id") Integer id);

    @Query(value = "select * from t_book where id=(select max(id) from t_book)", nativeQuery = true)
    Book getMaxIdBook();

    // @Modifying 注解、 @Transactional 注解
    @Query(value = "insert into t_book(name,author) values(?1,?2)", nativeQuery = true)
    @Modifying
    @Transactional
    // 参数要按顺序
    Integer addBook(String name, String author);

    @Query(value = "insert into t_book(name,author) values(:name,:author)", nativeQuery = true)
    @Modifying
    @Transactional
    // 使用 @Param ，参数可以不按顺序（推荐）
    Integer addBook2(@Param("name") String name, @Param("author") String author);
}
```

接口类说明：

- BookDao 接口继承自 `JpaRepository` ， JpaRepository 提供了一些基本的数据操作方法，例如：保存/更新/删除/列表查询/分页列表查询等。
- 在 BookDao 接口中也可以自己声明相关的方法，只需要**方法名称符合规范**。在 Spring Data 中，只要按照既定的规范命名方法，Spring Data Jpa 就知道你想干嘛，这样就不用写 SQL 了。相关规范参考下图：

![](https://oscimg.oschina.net/oscnet/up-aa6f3389fe8910fb1cf55b0abc44ecd7b20.png)

- 如果有特殊的查询，也可以自己定义方法名，使用 `@Query` 注解通过自定义 SQL 来实现。**默认情况下，在注解中使用的查询语言不是 SQL ，而是 JPQL** ，这是一种与数据库平台无关的面向对象的查询语言，有点定位类似于 Hibernate 中的 HQL 。当然可以通过在 `@Query` 注解中设置 `nativeQuery = true` 来表示使用原生查询，即大家所熟悉的 SQL 。
- 如果某个方法中的 SQL 涉及到数据操作，则需要使用 `@Modifying` 注解。

## 4 测试

最后在测试类中注入 `bookDao` 完成测试，如下：

```java
@SpringBootTest
class SpringBootJpaApplicationTests {

    @Autowired
    BookDao bookDao;

    @Test
    public void save() {
        Book book = new Book();
        book.setName("三国演义");
        book.setAuthor("罗贯中");
        bookDao.save(book);
    }

    @Test
    public void update() {
        Book book = new Book();
        book.setId(1);
        book.setAuthor("三国演义2");
        book.setName("罗贯中2");
        bookDao.saveAndFlush(book);
    }

    @Test
    public void delete() {
        bookDao.deleteById(1);
    }

    @Test
    public void findById() {
        Optional<Book> byId = bookDao.findById(1);
        System.out.println(byId.get());
    }

    @Test
    public void findAll() {
        List<Book> all = bookDao.findAll();
        System.out.println(all);
    }

    @Test
    public void findAllSort() {
        List<Book> list = bookDao.findAll(Sort.by(Sort.Direction.DESC, "id"));
        System.out.println(list);
    }

    @Test
    public void findAllPage() {
        Pageable pageable = PageRequest.of(0, 2);
        Page<Book> page = bookDao.findAll(pageable);
        System.out.println("总记录数：" + page.getTotalElements());
        System.out.println("当前页记录数：" + page.getNumberOfElements());
        System.out.println("每页记录数：" + page.getSize());
        System.out.println("获取总页数：" + page.getTotalPages());
        System.out.println("查询结果：" + page.getContent());
        System.out.println("当前页（从0开始计）" + page.getNumber());
        System.out.println("是否为首页：" + page.isFirst());
        System.out.println("是否为尾页：" + page.isLast());
    }

    @Test
    public void findBookById() {
        Book book = bookDao.findBookById(1);
        System.out.println(book);
    }

    @Test
    public void findByIdAndName() {
        Book book = bookDao.findByIdAndName(1, "三国演义");
        System.out.println(book);
    }

    @Test
    public void findByIdGreaterThan() {
        List<Book> list = bookDao.findByIdGreaterThan(2);
        System.out.println(list);
    }

    @Test
    public void findByIdLessThanOrNameContaining() {
        List<Book> list1 = bookDao.findByIdLessThanOrNameContaining(2, "花");
        System.out.println(list1);
    }

    @Test
    public void getById() {
        Book book = bookDao.getById(1);
        System.out.println(book);
    }

    @Test
    public void getBookById() {
        Book book = bookDao.getBookById(1);
        System.out.println(book);
    }

    @Test
    public void getByParam() {
        List<Book> list1 = bookDao.getByParam(1, "朝花夕拾");
        System.out.println(list1);
    }

    @Test
    public void getByParam2() {
        List<Book> list1 = bookDao.getByParam2("朝花夕拾", 1);
        System.out.println(list1);
    }

    @Test
    public void getMaxIdBook() {
        Book book = bookDao.getMaxIdBook();
        System.out.println(book);
    }


    @Test
    public void addBook() {
        Integer r1 = bookDao.addBook("朝花夕拾", "鲁迅");
        System.out.println(r1);
    }

    @Test
    public void addBook2() {
        Integer r2 = bookDao.addBook2("呐喊", "鲁迅");
        System.out.println(r2);
    }

}
```

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-dao/spring-boot-jpa](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-dao/spring-boot-jpa)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)