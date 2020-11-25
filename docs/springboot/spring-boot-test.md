---
title: Spring Boot 单元测试
date: 2019-11-07 09:56:53
categories: Spring Boot
tags: [Spring Boot, 单元测试]
toc: true
---
学习 Spring Boot 项目中的单元测试，实现 Service/Controller/JSON 测试。
<!-- more -->

## 1 准备工作

在 src/main/java 下相应的包中新建 Book 类，如下：

```java
public class Book {
    private Integer id;
    private String name;
    private String author;

    // getter/setter
}
```

在 src/main/java 下相应的包中新建 HelloService 类，如下：

```java
@Service
public class HelloService {
    public String sayHello(String name) {
        return "hello " + name;
    }
}
```

在 src/main/java 下相应的包中新建 HelloController 类，如下：

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello(String name) {
        return "hello " + name;
    }

    @PostMapping("/book")
    public Book addBook(@RequestBody Book book) {
        return book;
    }
}
```

## 2 Service 测试

在 src/test/java 下相应的包中新建 TestService 测试类，如下：

```java
// @RunWith(SpringRunner.class)
@SpringBootTest
public class TestService {

    @Autowired
    HelloService helloService;

    @Test
    public void contextLoads() {
        String hello = helloService.sayHello("cxy35");
        Assert.assertThat(hello, Matchers.is("hello cxy35"));
    }
}
```

## 3 Controller 测试

通过模拟 web 环境和请求来实现，支持 get/post/... 等请求方式。

在 src/test/java 下相应的包中新建 TestController 测试类，如下：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestController {

    @Autowired
    WebApplicationContext wac;

    MockMvc mockMvc;

    @Test
    public void contextLoads() {
    }

    @Before
    public void before() {
        mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
    }

    @Test
    public void testGet() throws Exception {
        MvcResult mvcResult = mockMvc.perform(
                MockMvcRequestBuilders.get("/hello")
                        .contentType(MediaType.APPLICATION_FORM_URLENCODED)
                        .param("name", "cxy35"))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andDo(MockMvcResultHandlers.print())
                .andReturn();
        System.out.println(mvcResult.getResponse().getContentAsString());
    }

    @Test
    public void testPost() throws Exception {
        Book book = new Book();
        book.setId(99);
        book.setName("三国演义");
        book.setAuthor("罗贯中");
        String s = new ObjectMapper().writeValueAsString(book);
        MvcResult mvcResult = mockMvc.perform(MockMvcRequestBuilders.post("/book").contentType(MediaType.APPLICATION_JSON).content(s))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andReturn();
        System.out.println(mvcResult.getResponse().getContentAsString());
    }
}
```

在 src/test/java 下相应的包中新建 TestController2 测试类，通过模板实现，如下：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.DEFINED_PORT)
public class TestController2 {
    @Autowired
    TestRestTemplate testRestTemplate;

    @Test
    public void contextLoads() {
        String cxy35 = testRestTemplate.getForObject("/hello?name={1}", String.class, "cxy35");
        System.out.println(cxy35);
    }
}
```

## 4 JSON 测试

在 src/test/java 下相应的包中新建 book.json 测试数据，如下：

```json
{"id":99,"name":"红楼梦","author":"曹雪芹"}
```

在 src/test/java 下相应的包中新建 TestJson 测试类，如下：

```java
@RunWith(SpringRunner.class)
// @SpringBootTest
@org.springframework.boot.test.autoconfigure.json.JsonTest
public class TestJson {
    @Autowired
    JacksonTester<Book> jacksonTester;

    @Test
    public void test() throws IOException {
        // 序列化
        Book book = new Book();
        book.setId(99);
        book.setName("红楼梦");
        book.setAuthor("曹雪芹");
        Assertions.assertThat(jacksonTester.write(book))
                .isEqualToJson("book.json");
        Assertions.assertThat(jacksonTester.write(book))
                .hasJsonPathStringValue("@.name");
        Assertions.assertThat(jacksonTester.write(book))
                .extractingJsonPathStringValue("@.name")
                .isEqualTo("红楼梦");
    }

    @Test
    public void test2() throws IOException {
        // 反序列化
        String content = "{\"id\":99,\"name\":\"红楼梦\",\"author\":\"曹雪芹\"}";
        Assertions.assertThat(jacksonTester.parseObject(content).getName()).isEqualTo("红楼梦");
    }
}
```

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-test](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-test)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)