学习在 Spring Boot 中使用 Elasticsearch。在 Spring Boot 中，使用的 Elasticsearch 实际上是 **`Spring Data Elasticsearch`** ， Spring Data 是 Spring 家族的一个子项目，用于简化 SQL 和 NoSQL 的访问，在 Spring Data 中，只要你的**方法名称符合规范**，它就知道你想干什么，不需要自己再去写 SQL 。
<!-- more -->

## 1 概述

详情见 [Elasticsearch 概述](https://www.cxy35.com/#/docs/elastic/elasticsearch-overview)

## 2 创建工程并配置

创建 Spring Boot 项目 `spring-boot-elasticsearch` ，添加 `Web/Elasticsearch` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-6ed70a0e1baefd59f7c55511d08df4dfc76.png)

最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
    </dependency>
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
</dependencies>
```

接着在 `application.properties` 配置文件中添加 Elasticsearch 的基本配置，如下：

```properties
spring.data.elasticsearch.repositories.enabled=true

spring.elasticsearch.rest.uris=http://localhost:9200
spring.elasticsearch.rest.username=elastic
spring.elasticsearch.rest.password=000000
```

## 3 使用

首先创建一个 `EsUser` 实体类，如下：

```java
@Document(indexName = "user", shards = 1, replicas = 0)
public class EsUser implements Serializable {
    private static final long serialVersionUID = -1L;
    @Id
    private Integer id;
    @Field(type = FieldType.Text, analyzer = "ik_max_word", searchAnalyzer = "ik_smart")
    private String username;
    @Field(type = FieldType.Keyword)
    private String password;
    @Field(type = FieldType.Boolean)
    private Boolean enabled = true;
    @Field(type = FieldType.Boolean)
    private Boolean locked = false;
    @Field(type = FieldType.Text, analyzer = "ik_max_word", searchAnalyzer = "ik_smart")
    private String address;
    @Field(type = FieldType.Text, analyzer = "ik_max_word", searchAnalyzer = "ik_smart")
    private String nickName;
    // @Field(type = FieldType.Date, format = DateFormat.date_hour_minute_second)
    // @JsonFormat(shape = JsonFormat.Shape.STRING, pattern ="yyyy-MM-dd'T'HH:mm:ss")
    private Date createTime;
    // @Field(type = FieldType.Date, format = DateFormat.date_hour_minute_second)
    // @JsonFormat(shape = JsonFormat.Shape.STRING, pattern ="yyyy-MM-dd'T'HH:mm:ss")
    private Date updateTime;

    // getter/setter
}
```

实体类说明：

- `@Document` 注解: 标识映射到 Elasticsearch 文档上的领域对象。
- `@Id` 注解: 标识文档的 id 。
- `@Field` 注解: 标识字段，可以指定各字段的类型、分析器等，最终会体现在对应 `index` 的 `mappings` 上。类型如下：

```java
public enum FieldType {
    Auto, // 自动判断，默认值
    Text, // 会进行分词
    Keyword, // 不会进行分词
    Long,
    Integer,
    Short,
    Byte,
    Double,
    Float,
    Half_Float,
    Scaled_Float,
    Date,
    Date_Nanos,
    Boolean,
    Binary,
    Integer_Range,
    Float_Range,
    Long_Range,
    Double_Range,
    Date_Range,
    Ip_Range,
    Object,
    Nested, // 嵌套对象
    Ip,
    TokenCount,
    Percolator,
    Flattened,
    Search_As_You_Type;

    private FieldType() {
    }
}
```

---

下面开始定义接口操作 Elasticsearch ，新增 `EsUserRepository` ，定义相关接口，如下：

```java
public interface EsUserRepository extends ElasticsearchRepository<EsUser, Integer> {
    // 方法定义规范：
    // 1.按照 Spring Data 的规范，查询方法以 find/get/read 开头
    // 2.涉及条件查询时，条件的属性用条件关键字连接，要注意的是：条件属性以首字母大写
    // 3.支持属性的级联查询. 若当前类有符合条件的属性, 则优先使用, 而不使用级联属性. 若需要使用级联属性, 则属性之间使用 _ 进行连接

    EsUser findByIdAndUsername(Integer id, String username);

    Page<EsUser> findByIdGreaterThan(Integer id, Pageable pageable);

    List<EsUser> findByIdLessThanOrUsernameContaining(Integer id, String username);

    // 使用 @Query 注解可以用 Elasticsearch 的 DSL 语句进行查询
    @Query("{\"bool\" : {\"must\" : {\"match\" : {\"address\" : \"?0\"}}}}")
    Page<EsUser> findByAddress(String address, Pageable pageable);
}
```

接口类说明：

- `EsUserRepository` 接口继承自 `ElasticsearchRepository` ， ElasticsearchRepository 提供了一些基本的数据操作方法，例如：保存/更新/删除/列表查询/分页列表查询等。
- 在 `EsUserRepository` 接口中也可以自己声明相关的方法，只需要**方法名称符合规范**。在 Spring Data 中，只要按照既定的规范命名方法，Spring Data Elasticsearch 就知道你想干嘛，这样就不用写 `DSL` 语句了。相关规范参考下图：

![](https://oscimg.oschina.net/oscnet/up-77c2a0d12eed84b15a2f7575a5e0295704d.png)

- 如果有特殊的查询，也可以自己定义方法名，使用 `@Query` 注解通过自定义 `DSL` 语句来实现。

这时启动 Spring Boot 项目，会自动创建一个名为 `user` 的索引。

## 4 测试

最后在测试类中注入 `esUserRepository` 完成测试，如下：

```java
@SpringBootTest
class SpringBootElasticsearchApplicationTests {

    @Autowired
    EsUserRepository esUserRepository;
    // 用于自定义复杂查询
    @Autowired
    private ElasticsearchRestTemplate elasticsearchRestTemplate;

    @Test
    public void save() {
        EsUser esUser = new EsUser();
        esUser.setId(1);
        esUser.setUsername("zhangsan");
        esUser.setPassword("123456");
        esUser.setAddress("浙江杭州");
        esUser.setNickName("张三");
        esUser.setCreateTime(new Date());
        esUser.setUpdateTime(new Date());
        EsUser esUserResult = esUserRepository.save(esUser);
        System.out.println(esUserResult);
    }

    @Test
    public void saveAll() {
        List<EsUser> esUserList = new ArrayList<>();
        for (int i = 2; i <= 11; i++) {
            EsUser esUser = new EsUser();
            esUser.setId(i);
            esUser.setUsername("lisi" + i);
            esUser.setPassword("123456");
            esUser.setAddress("浙江宁波");
            esUser.setNickName("李四" + i);
            esUser.setCreateTime(new Date());
            esUser.setUpdateTime(new Date());
            esUserList.add(esUser);
        }
        Iterable<EsUser> list = esUserRepository.saveAll(esUserList);
        System.out.println(list);
    }

    @Test
    public void deleteById() {
        esUserRepository.deleteById(5);
    }

    @Test
    public void delete() {
        EsUser esUser = new EsUser();
        esUser.setId(6);
        esUserRepository.delete(esUser);
    }

    @Test
    public void deleteAll() {
        esUserRepository.deleteAll();
    }

    @Test
    public void findById() {
        Optional<EsUser> esUser = esUserRepository.findById(1);
        System.out.println(esUser.get());
    }

    @Test
    public void findAllById() {
        List<Integer> idList = new ArrayList<>();
        idList.add(1);
        idList.add(2);
        Iterable<EsUser> list = esUserRepository.findAllById(idList);
        System.out.println(list);
    }

    @Test
    public void findAll() {
        Iterable<EsUser> list = esUserRepository.findAll();
        System.out.println(list);
    }

    @Test
    public void findAllSort() {
        Iterable<EsUser> list = esUserRepository.findAll(Sort.by(Sort.Direction.DESC, "id"));
        System.out.println(list);
    }

    @Test
    public void findAllPage() {
        Pageable pageable = PageRequest.of(0, 2);
        Page<EsUser> page = esUserRepository.findAll(pageable);
        System.out.println("总记录数：" + page.getTotalElements());
        System.out.println("当前页记录数：" + page.getNumberOfElements());
        System.out.println("每页记录数：" + page.getSize());
        System.out.println("总页数：" + page.getTotalPages());
        System.out.println("查询结果：" + page.getContent());
        System.out.println("当前页（从0开始计）：" + page.getNumber());
        System.out.println("是否为首页：" + page.isFirst());
        System.out.println("是否为尾页：" + page.isLast());
    }

    /**
     * more_like_this query
     */
    @Test
    public void searchSimilar() {
        EsUser esUser = new EsUser();
        esUser.setId(2);
        Pageable pageable = PageRequest.of(0, 2);
        Page<EsUser> page = esUserRepository.searchSimilar(esUser, new String[]{"address"}, pageable);
        System.out.println("总记录数：" + page.getTotalElements());
        System.out.println("当前页记录数：" + page.getNumberOfElements());
        System.out.println("每页记录数：" + page.getSize());
        System.out.println("总页数：" + page.getTotalPages());
        System.out.println("查询结果：" + page.getContent());
        System.out.println("当前页（从0开始计）：" + page.getNumber());
        System.out.println("是否为首页：" + page.isFirst());
        System.out.println("是否为尾页：" + page.isLast());
    }

    @Test
    public void existsById() {
        boolean b = esUserRepository.existsById(1);
        System.out.println(b);
    }

    @Test
    public void count() {
        long count = esUserRepository.count();
        System.out.println(count);
    }

    /*=============== 自定义简单查询-开始 ===============*/
    @Test
    public void findByIdAndUsername() {
        EsUser esUser = esUserRepository.findByIdAndUsername(2, "lisi");
        System.out.println(esUser);
    }

    @Test
    public void findByIdGreaterThan() {
        Pageable pageable = PageRequest.of(0, 2);
        Page<EsUser> page = esUserRepository.findByIdGreaterThan(4, pageable);
        System.out.println("总记录数：" + page.getTotalElements());
        System.out.println("当前页记录数：" + page.getNumberOfElements());
        System.out.println("每页记录数：" + page.getSize());
        System.out.println("总页数：" + page.getTotalPages());
        System.out.println("查询结果：" + page.getContent());
        System.out.println("当前页（从0开始计）：" + page.getNumber());
        System.out.println("是否为首页：" + page.isFirst());
        System.out.println("是否为尾页：" + page.isLast());
    }

    @Test
    public void findByIdLessThanOrUsernameContaining() {
        List<EsUser> list = esUserRepository.findByIdLessThanOrUsernameContaining(10, "si");
        System.out.println(list);
    }

    @Test
    public void findByAddress() {
        Pageable pageable = PageRequest.of(0, 2);
        Page<EsUser> page = esUserRepository.findByAddress("宁波", pageable);
        System.out.println("总记录数：" + page.getTotalElements());
        System.out.println("当前页记录数：" + page.getNumberOfElements());
        System.out.println("每页记录数：" + page.getSize());
        System.out.println("总页数：" + page.getTotalPages());
        System.out.println("查询结果：" + page.getContent());
        System.out.println("当前页（从0开始计）：" + page.getNumber());
        System.out.println("是否为首页：" + page.isFirst());
        System.out.println("是否为尾页：" + page.isLast());
    }
    /*=============== 自定义简单查询-结束 ===============*/

    /*=============== 自定义复杂查询（ElasticsearchRestTemplate）-开始 ===============*/
    /**
     * 根据关键字搜索用户名或昵称，再增加过滤、聚合、排序、分页
     */
    @Test
    public void search() {
        Boolean enabled = true;
        Boolean locked = false;
        String keyword = "四";
        Integer sort = 1;
        PageImpl<EsUser> page = null;
        NativeSearchQueryBuilder nativeSearchQueryBuilder = new NativeSearchQueryBuilder();

        // 搜索
        if (StringUtils.isEmpty(keyword)) {
            nativeSearchQueryBuilder.withQuery(QueryBuilders.matchAllQuery());
        } else {
            // nativeSearchQueryBuilder.withQuery(QueryBuilders.multiMatchQuery(keyword, "username", "nickName"));

            List<FunctionScoreQueryBuilder.FilterFunctionBuilder> filterFunctionBuilders = new ArrayList<>();
            filterFunctionBuilders.add(new FunctionScoreQueryBuilder.FilterFunctionBuilder(QueryBuilders.matchQuery("username", keyword),
                    ScoreFunctionBuilders.weightFactorFunction(10)));
            filterFunctionBuilders.add(new FunctionScoreQueryBuilder.FilterFunctionBuilder(QueryBuilders.matchQuery("nickName", keyword),
                    ScoreFunctionBuilders.weightFactorFunction(2)));
            FunctionScoreQueryBuilder.FilterFunctionBuilder[] builders = new FunctionScoreQueryBuilder.FilterFunctionBuilder[filterFunctionBuilders.size()];
            filterFunctionBuilders.toArray(builders);
            FunctionScoreQueryBuilder functionScoreQueryBuilder = QueryBuilders.functionScoreQuery(builders)
                    .scoreMode(FunctionScoreQuery.ScoreMode.SUM)
                    .setMinScore(2);
            nativeSearchQueryBuilder.withQuery(functionScoreQueryBuilder);
        }

        // 过滤
        if (enabled != null || locked != null) {
            BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
            if (enabled != null) {
                boolQueryBuilder.must(QueryBuilders.termQuery("enabled", enabled));
            }
            if (locked != null) {
                boolQueryBuilder.must(QueryBuilders.termQuery("locked", locked));
            }
            nativeSearchQueryBuilder.withFilter(boolQueryBuilder);
        }

        // 聚合，对应的字段一般设置为 FieldType.Keyword
        // nativeSearchQueryBuilder.addAggregation(AggregationBuilders.terms("passwords").field("password"));
        // nativeSearchQueryBuilder.addAggregation(AggregationBuilders.terms("addresses").field("address"));

        // 排序
        if (sort == 1) {
            // 按id降序
            nativeSearchQueryBuilder.withSort(SortBuilders.fieldSort("id").order(SortOrder.DESC));
        } else if (sort == 2) {
            // 按更新时间降序
            nativeSearchQueryBuilder.withSort(SortBuilders.fieldSort("updateTime").order(SortOrder.DESC));
        } else if (sort == 3) {
            // 按地址升序
            nativeSearchQueryBuilder.withSort(SortBuilders.fieldSort("address").order(SortOrder.ASC));
        } else {
            // 按相关度
            nativeSearchQueryBuilder.withSort(SortBuilders.scoreSort().order(SortOrder.DESC));
        }

        // 分页
        Pageable pageable = PageRequest.of(0, 2);
        nativeSearchQueryBuilder.withPageable(pageable);

        NativeSearchQuery searchQuery = nativeSearchQueryBuilder.build();
        System.out.println("DSL: " + searchQuery.getQuery().toString());
        SearchHits<EsUser> searchHits = elasticsearchRestTemplate.search(searchQuery, EsUser.class);
        if (searchHits.getTotalHits() <= 0) {
            page = new PageImpl<>(Collections.emptyList(), pageable, 0);
        }
        List<EsUser> esUserList = searchHits.stream().map(SearchHit::getContent).collect(Collectors.toList());
        page = new PageImpl<>(esUserList, pageable, searchHits.getTotalHits());
        System.out.println("总记录数：" + page.getTotalElements());
        System.out.println("当前页记录数：" + page.getNumberOfElements());
        System.out.println("每页记录数：" + page.getSize());
        System.out.println("总页数：" + page.getTotalPages());
        System.out.println("查询结果：" + page.getContent());
        System.out.println("当前页（从0开始计）：" + page.getNumber());
        System.out.println("是否为首页：" + page.isFirst());
        System.out.println("是否为尾页：" + page.isLast());
    }
    /*=============== 自定义复杂查询（ElasticsearchRestTemplate）-结束 ===============*/
}
```

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-dao/spring-boot-elasticsearch](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-dao/spring-boot-elasticsearch)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)