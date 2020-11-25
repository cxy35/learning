---
title: Spring Boot 整合 Swagger2
date: 2019-12-22 14:29:21
categories: Spring Boot
tags: [Spring Boot, Swagger2]
toc: true
---
学习在 Spring Boot 中使用 Swagger2 实时生成在线接口文档，还支持接口测试。特别是在前后端分离开发时，可以说是一大神器。
<!-- more -->

## 1 创建工程

### 1.1 手动集成（老版本，无 Starter）

创建 Spring Boot 项目 `spring-boot-swagger2` ，添加 `Web` 依赖。之后手动在 pom 文件中添加 Swagger2 相关的两个依赖，最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger2</artifactId>
        <version>2.9.2</version>
    </dependency>
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>2.9.2</version>
    </dependency>

    <!-- 1.5.21版本覆盖默认的1.5.20版本，解决@ApiModelProperty的example默认值类型转换异常BUG - java.lang.NumberFormatException: For input string: "" -->
    <dependency>
        <groupId>io.swagger</groupId>
        <artifactId>swagger-annotations</artifactId>
        <version>1.5.21</version>
    </dependency>
    <dependency>
        <groupId>io.swagger</groupId>
        <artifactId>swagger-models</artifactId>
        <version>1.5.21</version>
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

### 1.2 自动集成（新版本，使用 Starter）

> 注：springfox 3.0.0 版本开始已经有对应的官方 Spring Boot Starter，在 Spring Boot 项目中使用就更方便了。

创建 Spring Boot 项目 `spring-boot-swagger2` ，添加 `Web` 依赖。之后手动在 pom 文件中添加 Swagger2 相关的两个依赖，最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-boot-starter</artifactId>
        <version>3.0.0</version>
    </dependency>
    
    <!-- 1.5.21版本覆盖默认的1.5.20版本，解决@ApiModelProperty的example默认值类型转换异常BUG - java.lang.NumberFormatException: For input string: "" -->
    <dependency>
        <groupId>io.swagger</groupId>
        <artifactId>swagger-annotations</artifactId>
        <version>1.5.21</version>
    </dependency>
    <dependency>
        <groupId>io.swagger</groupId>
        <artifactId>swagger-models</artifactId>
        <version>1.5.21</version>
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

## 2 Swagger2 配置

新增 `Swagger2Config` 配置类，如下：

```java
@Configuration
@EnableSwagger2 // 启用 Swagger2
public class Swagger2Config {
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .pathMapping("/")
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.cxy35.sample.springboot.swagger2.controller"))
                .paths(PathSelectors.any())
                .build().apiInfo(new ApiInfoBuilder()
                        .title("这是网站的标题...")
                        .description("这是网站的描述...")
                        .version("v1.0")
                        .contact(new Contact("这是联系人名称", "https://cxy35.com", "123456@qq.com"))
                        .license("这是网站使用的协议...")
                        .licenseUrl("https://www.baidu.com")
                        .build());
    }
}
```
启动项目，访问 [http://127.0.0.1:8080/swagger-ui.html](http://127.0.0.1:8080/swagger-ui.html)（老版本） 或 [http://127.0.0.1:8080/swagger-ui/](http://127.0.0.1:8080/swagger-ui/)（新版本），查看效果如下：

![](https://oscimg.oschina.net/oscnet/up-d0d26d6bdd1173181d9d07b7e0eac16e5de.png)

> 注：在新版本中，支持在配置文件中完成一些配置，比如：`springfox.documentation.enabled` 配置可以控制是否启用 Swagger 文档生成功能（一般在开发环境开启，在生产环境关闭）。

```properties
#springfox.documentation.enabled=false
```

其他配置如下：

![](https://oscimg.oschina.net/oscnet/up-9120c5f8cc878c26f6646dbbd761b84f76d.png)

## 3 使用

新增 `UserController` 测试接口，如下：

```java
@RestController
@Api(tags = "用户管理接口")
public class UserController {
    @GetMapping("/user")
    @ApiOperation(value = "查询用户", notes = "根据用户id查询用户")
    @ApiImplicitParam(name = "id", value = "用户id", required = true, defaultValue = "99")
    public User getUserById(Integer id) {
        User user = new User();
        user.setId(id);
        user.setUsername("cxy35");
        user.setAddress("HZ");
        return user;
    }

    @PutMapping("/user")
    @ApiOperation(value = "更新用户", notes = "根据用户id更新用户名")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "id", value = "用户id", required = true, defaultValue = "99"),
            @ApiImplicitParam(name = "username", value = "用户名", required = true, defaultValue = "cxy35")
    })
    // @ApiIgnore
    public User updateUsernameById(String username, Integer id) {
        User user = new User();
        user.setId(id);
        user.setUsername(username);
        return user;
    }

    @PostMapping("/user")
    @ApiOperation(value = "添加用户", notes = "添加用户接口")
    public User addUser(@RequestBody User user) {
        return user;
    }

    @DeleteMapping("/user/{id}")
    @ApiOperation(value = "删除用户", notes = "根据用户id删除用户")
    @ApiImplicitParam(name = "id", value = "用户id", required = true, defaultValue = "99")
    @ApiResponses({
            @ApiResponse(code = 200, message = "删除成功"),
            @ApiResponse(code = 500, message = "删除失败")
    })
    public void deleteUserById(@PathVariable Long id) {

    }

    @GetMapping("/hello")
    public String hello(String name) {
        return "hello " + name + " !";
    }
}
```

注解说明：

- `@Api` 注解：用来描述一个 Controller 类。
- `@ApiOperation` 注解：用来描述一个接口。
- `@ApiImplicitParam` 注解：用来描述一个参数，可以配置参数的中文含义，也可以给参数设置默认值，这样在接口测试的时候可以避免手动输入。
- `@ApiImplicitParams` 注解：如果有多个参数，则需要使用多个 @ApiImplicitParam 注解来描述，多个 @ApiImplicitParam 注解需要放在一个 @ApiImplicitParams 中。
- 需要注意的是， @ApiImplicitParam 注解中虽然可以指定参数是必填的，但是却不能代替 @RequestParam(required = true) ，前者的必填只是在 Swagger2 框架内必填，抛弃了 Swagger2 ，这个限制就没用了，所以假如开发者需要指定一个参数必填， @RequestParam(required = true) 注解还是不能省略。
- 如果参数是一个对象（例如上文的 addUser 接口），对于参数的描述也可以放在实体类中，比如：

```java
@ApiModel(value = "用户实体类",description = "用户信息描述类")
public class User {
    @ApiModelProperty(value = "用户id")
    private Integer id;
    @ApiModelProperty(value = "用户名")
    private String username;
    @ApiModelProperty(value = "用户地址")
    private String address;

    // getter/setter
}
```

注解说明：

- `@ApiModel` 注解：用来描述一个实体类。
- `@ApiModelProperty` 注解：用来描述一个实体类的字段。

重新启动项目，访问 [http://127.0.0.1:8080/swagger-ui.html](http://127.0.0.1:8080/swagger-ui.html)（老版本） 或 [http://127.0.0.1:8080/swagger-ui/](http://127.0.0.1:8080/swagger-ui/)（新版本），查看效果如下：

![](https://oscimg.oschina.net/oscnet/up-a18f94d0dfc340f2f729dd2067003bcd957.png)

![](https://oscimg.oschina.net/oscnet/up-a53a315ae448131865eb8a57745e717267f.png)

![](https://oscimg.oschina.net/oscnet/up-524beb1f190d212bd7dde7d132f9e6cefca.png)

![](https://oscimg.oschina.net/oscnet/up-37da2f7d0c1aeb300c51930bf8a4fc41bec.png)

![](https://oscimg.oschina.net/oscnet/up-4349fe738d12770e8de6b431b93a488cfa8.png)

## 4 Spring Security 中的配置

如果项目中集成了 Spring Security ，默认情况下 Swagger2 文档可能会被拦截，需要在 Spring Security 的配置类中重写 configure 方法，增加如下过滤配置：

```java
@Override
public void configure(WebSecurity web) throws Exception {
    web.ignoring()
            .antMatchers("/swagger-ui.html")
            .antMatchers("/v2/**")
            .antMatchers("/swagger-resources/**");
}
```

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-swagger2](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-swagger2)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)