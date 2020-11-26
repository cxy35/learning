学习在 Spring Cloud 中使用 Hystrix 实现断路器，包括服务降级/容错、异步调用、异常处理、请求缓存、请求合并等功能。它是 Netflix 家族成员之一。
<!-- more -->

## 1 概述

Hystrix 叫做**断路器/熔断器**。微服务系统中，整个系统出错的概率非常高，因为在微服务系统中，涉及到的模块太多了，每一个模块出错，都有可能导致整个服务出错，只有当所有模块都稳定运行时，整个服务才算是稳定运行。

我们希望当整个系统中，某一个模块无法正常工作时，能够通过我们提前配置的一些东西，来使得整个系统正常运行，即单个模块出问题，不影响整个系统。

## 2 准备工作

### 2.1 服务注册

创建 Spring Boot 项目 `hystrix-client-provider` ，作为我们的**服务提供者**，添加 `Web/Eureka Client` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-0ccb137be7b2d3ed22cc9288f7a31927aa4.png)

最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
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

---

项目创建成功后，修改 `application.properties` 配置文件，将 hystrix-client-provider 注册到 Eureka Server 上（服务注册中心使用 Eureka Server ），如下：

```properties
# 当前服务的名称
spring.application.name=hystrix-client-provider
# 当前服务的端口
server.port=3000

# 服务注册中心地址
eureka.client.service-url.defaultZone=http://127.0.0.1:1111/eureka
```

接下来，启动 Eureka Server ，待服务注册中心启动成功后，再启动 hystrix-client-provider ，两者都启动成功后，访问 [http://127.0.0.1:1111](http://127.0.0.1:1111) 可以看到 hystrix-client-provider 的注册信息。

![](https://oscimg.oschina.net/oscnet/up-2850f23837278045d7f70641f91cd9f7006.png)

---

当然 hystrix-client-provider 也可以集群化部署，下面对 hystrix-client-provider 进行打包，之后我们在命令行启动两个 provider 实例：
 
```bash
java -jar hystrix-client-provider-0.0.1-SNAPSHOT.jar --server.port=3000
java -jar hystrix-client-provider-0.0.1-SNAPSHOT.jar --server.port=3001
```

---

最后在 hystrix-client-provider 提供一个 hello 接口，用于后续服务消费者 hystrix-client-consumer 来消费，如下：

```java
@RestController
public class ProviderController {
    @Value("${server.port}")
    Integer port; // 支持启动多个实例，做负载均衡，用端口区分

    @GetMapping("/hello")
    public String hello() {
        return "hello cxy35: " + port;
    }
}
```

### 2.2 服务消费

创建 Spring Boot 项目 `hystrix-client-consumer` ，作为我们的**服务消费者**，添加 `Web/Eureka Client/Hystrix` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-0495818e2972dda2792df642dbf8bde4257.png)

最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
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

---

项目创建成功后，修改 `application.properties` 配置文件，将 hystrix-client-consumer 注册到 Eureka Server 上（服务注册中心使用 Eureka Server ），如下：

```properties
# 当前服务的名称
spring.application.name=hystrix-client-consumer
# 当前服务的端口
server.port=3002

# 服务注册中心地址
eureka.client.service-url.defaultZone=http://127.0.0.1:1111/eureka
```

接着，在项目启动类上添加 `@EnableCircuitBreaker` 注解，开启断路器功能，并添加 RestTemplate ，如下：

```java
@SpringBootApplication
@EnableCircuitBreaker // 开启断路器的功能
// @SpringCloudApplication // 组合注解
public class HystrixClientConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(HystrixClientConsumerApplication.class, args);
    }

    @Bean
    @LoadBalanced
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

接下来，启动 hystrix-client-consumer ，访问 [http://127.0.0.1:1111](http://127.0.0.1:1111) 可以看到 hystrix-client-consumer 的注册信息。

![](https://oscimg.oschina.net/oscnet/up-0dcaec925feaa99838e2ded33b0771febb9.png)

---

最后在 hystrix-client-consumer 中新增测试业务类和接口，去实现服务调用，从而消费 hystrix-client-provider 中提供的接口，如下：

> 约定：本文中的服务调用失败（测试服务降级/容错），可以采用关闭某个 hystrix-client-provider 来模拟，短时间内会报错（因为 provider 地址会缓存 consumer 上一段时间），从而达到我们的目的。

## 3 服务降级/容错

1. 注解方式

新建测试业务类 `ConsumerService` ，如下：

```java
@Service
public class ConsumerService {
    @Autowired
    RestTemplate restTemplate;

    /**
     * 在这个方法中，我们将发起一个远程调用，去调用 hystrix-client-provider 中提供的 /hello 接口
     * <p>
     * 但是，这个调用可能会失败，可以采用关闭某个 hystrix-client-provider 来模拟。
     * <p>
     * 我们在这个方法上添加 @HystrixCommand 注解，配置 fallbackMethod 属性，这个属性表示该方法调用失败时的临时替代方法
     *
     * @return
     */
    // 服务降级/容错
    @HystrixCommand(fallbackMethod = "error")
    public String testHystrix() {
        return restTemplate.getForObject("http://hystrix-client-provider/hello", String.class);
    }

    /**
     * 实现服务容错/降级：这个方法就是请求失败的回调
     * <p>
     * 注意：这个方法名字要和上述 fallbackMethod 中指定的一致，方法返回值也要和对应的方法一致
     *
     * @return
     */
    // 服务降级/容错，这里简单实现
    public String error() {
        return "error";
    }
}
```

---

新建测试接口 `ConsumerController` ，如下：

```java
@RestController
public class ConsumerController {
    @Autowired
    ConsumerService consumerService;
    @Autowired
    RestTemplate restTemplate;

    // 服务降级/容错
    @GetMapping("/testHystrix")
    public String testHystrix() {
        return consumerService.testHystrix();
    }
}
```

访问 [http://127.0.0.1:3002/testHystrix](http://127.0.0.1:3002/testHystrix) 完成测试。

2. 请求命令方式

请求命令方式就是以**继承类**的方式来替代前面的注解方式。

新建测试业务类 `ConsumerService2` ，如下：

```java
public class ConsumerService2 extends HystrixCommand<String> {

    RestTemplate restTemplate;

    public ConsumerService2(Setter setter, RestTemplate restTemplate) {
        super(setter);
        this.restTemplate = restTemplate;
    }

    // 服务降级/容错
    @Override
    protected String run() throws Exception {
        return restTemplate.getForObject("http://hystrix-client-provider/hello", String.class);
    }

    // 服务降级/容错，这个方法就是请求失败的回调
    @Override
    protected String getFallback() {
        return "error2";
    }
}
```

---

新建测试接口 `ConsumerController2` ，如下：

```java
@RestController
public class ConsumerController2 {
    @Autowired
    RestTemplate restTemplate;

    // 服务降级/容错
    @GetMapping("/testHystrix2")
    public String testHystrix2() {
        // 1. 直接执行
        ConsumerService2 command = new ConsumerService2(HystrixCommand.Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("cxy35")), restTemplate);
        return command.execute();
    }
}
```

访问 [http://127.0.0.1:3002/testHystrix2](http://127.0.0.1:3002/testHystrix2) 完成测试。

**注意：一个 HystrixCommand 实例只能执行一次。**

## 4 异步调用

1. 注解方式

修改测试业务类 `ConsumerService` ，增加方法，如下：

```java
// 异步调用
@HystrixCommand(fallbackMethod = "error")
public Future<String> testHystrixAsync() {
    return new AsyncResult<String>() {
        @Override
        public String invoke() {
            return restTemplate.getForObject("http://hystrix-client-provider/hello", String.class);
        }
    };
}
```

---

修改测试接口 `ConsumerController` ，增加方法，如下：

```java
// 异步调用
@GetMapping("/testHystrixAsync")
public String testHystrixAsync() {
    Future<String> future = consumerService.testHystrixAsync();
    try {
        return future.get();
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
    return "";
}
```

访问 [http://127.0.0.1:3002/testHystrixAsync](http://127.0.0.1:3002/testHystrixAsync) 完成测试。

2. 请求命令方式

修改测试接口 `ConsumerController2` ，增加方法，如下：

```java
// 异步调用
@GetMapping("/testHystrixAsync2")
public String testHystrixAsync2() {
    // 2. 先入队，后执行
    ConsumerService2 command = new ConsumerService2(HystrixCommand.Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("cxy35")), restTemplate);
    try {
        Future<String> queue = command.queue();
        return queue.get();
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
    return "";
}
```

访问 [http://127.0.0.1:3002/testHystrixAsync2](http://127.0.0.1:3002/testHystrixAsync2) 完成测试。

## 5 异常处理

当发起服务调用时，如果不是 hystrix-client-provider 的原因导致请求调用失败，而是 hystrix-client-consumer 中本身代码有问题导致的请求失败，**即 hystrix-client-consumer 中抛出了异常，这个时候，也会自动进行服务降级**，只不过这个时候降级，我们还需要知道到底是哪里出异常了。

如下示例代码，执行时抛出异常，那么一样也会进行服务降级，进入到对应的 error 方法中，在 error 方法中，我们可以获取到异常的详细信息：

1. 注解方式

修改测试业务类 `ConsumerService` ，增加方法，如下：

```java
// 异常处理
@HystrixCommand(fallbackMethod = "error2")
public String testHystrixException() {
    int i = 1 / 0; // 抛异常，会自动进行服务降级
    return restTemplate.getForObject("http://hystrix-client-provider/hello", String.class);
}

// 异常处理
public String error2(Throwable t) {
    return "error: " + t.getMessage();
}
```

---

修改测试接口 `ConsumerController` ，增加方法，如下：

```java
// 异常处理
@GetMapping("/testHystrixException")
public String testHystrixException() {
    return consumerService.testHystrixException();
}
```

访问 [http://127.0.0.1:3002/testHystrixException](http://127.0.0.1:3002/testHystrixException) 完成测试。

---

另外，如果抛异常了，我们希望异常直接抛出，不要服务降级，那么只需要配置忽略某一个异常即可，如下：

```java
// 异常处理
@HystrixCommand(fallbackMethod = "error2", ignoreExceptions = ArithmeticException.class)
public String testHystrixException() {
    int i = 1 / 0;
    return restTemplate.getForObject("http://hystrix-client-provider/hello", String.class);
}
```

2. 请求命令方式

修改测试业务类 `ConsumerService2` ，修改方法，如下：

```java
// 异常处理
@Override
protected String run() throws Exception {
    int i = 1 / 0;
    return restTemplate.getForObject("http://hystrix-client-provider/hello", String.class);
}

// 异常处理
@Override
protected String getFallback() {
    return "error2: " + getExecutionException().getMessage();
}
```

---

修改测试接口 `ConsumerController2` ，修改方法，如下：

```java
// 异常处理
@GetMapping("/testHystrixException2")
public String testHystrixException2() {
    ConsumerService2 command = new ConsumerService2(HystrixCommand.Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("cxy35")), restTemplate);
    return command.execute();
}
```

访问 [http://127.0.0.1:3002/testHystrixException2](http://127.0.0.1:3002/testHystrixException2) 完成测试。

## 6 请求缓存

请求缓存就是在 hystrix-client-consumer 中调用同一个接口，如果参数相同，则可以使用之前缓存下来的数据。

首先在 `ProviderController` 中增加 /hello2 接口，如下：

```java
@GetMapping("/hello2")
public String hello2(String name) {
    System.out.println(new Date());
    return "hello " + name + ": " + port;
}
```

---

1. 注解方式

修改测试业务类 `ConsumerService` ，增加方法，如下：

```java
// 请求缓存
@HystrixCommand(fallbackMethod = "error3")
// 这个注解表示该方法的请求结果会被缓存起来
// 默认缓存的 key 为所有参数的值（可通过 @CacheKey 修改，如指定某一个参数），缓存的 value 为方法的返回值
@CacheResult
// 下面的配置，虽然有两个参数，但是缓存时以 name 为准。
// 也就是说，两次请求中，只要 name 一样，即使 age 不一样，第二次请求也可以使用第一次请求缓存的结果。
// public String testHystrixCache(@CacheKey String name, Integer age) {
public String testHystrixCache(String name) {
    return restTemplate.getForObject("http://hystrix-client-provider/hello2?name={1}", String.class, name);
}

// 请求缓存：删除数据库中的数据，同时删除缓存中的数据
@HystrixCommand
// 必须指定 commandKey 属性，commandKey 其实就是缓存方法的名字，指定了 commandKey，@CacheRemove 才能找到数据缓存在哪里了，进而才能成功删除掉数据。
@CacheRemove(commandKey = "testHystrixCache")
public String deleteUserByName(String name) {
    return null;
}

// 请求缓存
public String error3(String name) {
    return "error: " + name;
}
```

---

修改测试接口 `ConsumerController` ，增加方法，如下：

```java
// 请求缓存
@GetMapping("/testHystrixCache")
public void testHystrixCache() {
    // 开启缓存
    // 缓存默认不会生效，我们使用缓存，都有一个缓存生命周期这样一个概念。
    // 需要初始化 HystrixRequestContext，初始化完成后，缓存开始生效。close 之后，缓存失效。
    HystrixRequestContext ctx = HystrixRequestContext.initializeContext();

    // 第一请求完，数据已经缓存下来了
    String cxy35 = consumerService.testHystrixCache("cxy35");
    System.out.println(cxy35);

    // 删除数据，同时缓存中的数据也会被删除
    consumerService.deleteUserByName("cxy35");

    // 第二次请求时，直接使用缓存数据，不会再调用 provider 。除非中间调用了 deleteUserByName 清除掉缓存
    cxy35 = consumerService.testHystrixCache("cxy35");
    System.out.println(cxy35);

    // 关闭缓存
    // 在 ctx close 之前，缓存是有效的，close 之后，缓存就失效了。
    // 访问一次本接口，provider 只会被调用一次（第二次使用的缓存，除非中间调了清除缓存的接口，如 deleteUserByName）。
    ctx.close();
}
```

访问 [http://127.0.0.1:3002/testHystrixCache](http://127.0.0.1:3002/testHystrixCache) 完成测试，会发现 consumer 中调用了两次，而provider 中只打印了一次。

2. 请求命令方式

如果是继承的方式使用 Hystrix ，只需要重写 getCacheKey 方法即可。

修改测试业务类 `ConsumerService2` ，增加方法，如下：

```java
// 请求缓存
@Override
protected String run() throws Exception {
    return restTemplate.getForObject("http://hystrix-client-provider/hello2?name={1}", String.class, name);
}

// 请求缓存
@Override
protected String getFallback() {
    return "error2: " + getExecutionException().getMessage();
}

// 请求缓存
@Override
protected String getCacheKey() {
    return name;
}
```

---

修改测试接口 `ConsumerController2` ，增加方法，如下：

```java
// 请求缓存
@GetMapping("/testHystrixCache2")
public void testHystrixCache2() {
    // 开启缓存
    // 缓存默认不会生效，我们使用缓存，都有一个缓存生命周期这样一个概念。
    // 需要初始化 HystrixRequestContext，初始化完成后，缓存开始生效。close 之后，缓存失效。
    HystrixRequestContext ctx = HystrixRequestContext.initializeContext();

    // 第一请求完，数据已经缓存下来了
    ConsumerService2 command = new ConsumerService2(HystrixCommand.Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("cxy35")), restTemplate, "cxy35");
    String r = command.execute();
    System.out.println(r);

    // 第二次请求时，直接使用缓存数据，不会再调用 provider 。除非中间调用了 deleteUserByName 清除掉缓存
    command = new ConsumerService2(HystrixCommand.Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("cxy35")), restTemplate, "cxy35");
    r = command.execute();
    System.out.println(r);

    // 关闭缓存
    // 在 ctx close 之前，缓存是有效的，close 之后，缓存就失效了。
    // 访问一次本接口，provider 只会被调用一次（第二次使用的缓存，除非中间调了清除缓存的接口，如 deleteUserByName）。
    ctx.close();
}
```

访问 [http://127.0.0.1:3002/testHystrixCache2](http://127.0.0.1:3002/testHystrixCache2) 完成测试，会发现 consumer 中调用了两次，而provider 中只打印了一次。

## 7 请求合并

如果 hystrix-client-consumer 中，频繁的调用 hystrix-client-provider 中的同一个接口，在调用时，只是参数不一样，那么这样情况下，我们就可以将多个请求合并成一个，这样可以有效提高请求发送的效率。

首先我们在 hystrix-client-provider 的 `ProviderController` 中提供一个请求合并的接口，如下：

```java
// 假设 consumer 传过来的多个 id 的格式是 1,2,3,4....
@GetMapping("/user/{ids}")
public List<User> getUserByIds(@PathVariable String ids) {
    System.out.println(ids);
    String[] split = ids.split(",");
    List<User> users = new ArrayList<>();
    for (String s : split) {
        User u = new User();
        u.setId(Integer.parseInt(s));
        users.add(u);
    }
    return users;
}
```

---

1. 注解方式（简单，推荐）

修改测试业务类 `ConsumerService` ，增加方法，如下：

```java
// 请求合并，必须要用异步调用方式，并指定批处理的方法为 getUsersByIds
// 这里还配置了一个属性 timerDelayInMilliseconds 为 200 毫秒
@HystrixCollapser(batchMethod = "getUsersByIds", collapserProperties = {@HystrixProperty(name = "timerDelayInMilliseconds", value = "200")})
public Future<User> testHystrixCollapser(Integer id) {
    return null;
}

// 请求合并
@HystrixCommand
public List<User> getUsersByIds(List<Integer> ids) {
    User[] users = restTemplate.getForObject("http://hystrix-client-provider/user/{1}", User[].class, StringUtils.join(ids, ","));
    return Arrays.asList(users);
}
```

---

修改测试接口 `ConsumerController` ，增加方法，如下：

```java
// 请求合并
@GetMapping("/testHystrixCollapser")
public void testHystrixCollapser() throws ExecutionException, InterruptedException {
    HystrixRequestContext ctx = HystrixRequestContext.initializeContext();

    // 这3个请求会一起发起
    Future<User> q1 = consumerService.testHystrixCollapser(99);
    Future<User> q2 = consumerService.testHystrixCollapser(98);
    Future<User> q3 = consumerService.testHystrixCollapser(97);
    User u1 = q1.get();
    User u2 = q2.get();
    User u3 = q3.get();
    System.out.println(u1);
    System.out.println(u2);
    System.out.println(u3);

    Thread.sleep(2000);

    // 这个请求会单独发起
    Future<User> q4 = consumerService.testHystrixCollapser(96);
    User u4 = q4.get();
    System.out.println(u4);

    ctx.close();
}
```

访问 [http://127.0.0.1:3002/testHystrixCollapser](http://127.0.0.1:3002/testHystrixCollapser) 完成测试。

2. 请求命令方式

新增 `UserService` ，如下：

```java
@Service
public class UserService {
    @Autowired
    RestTemplate restTemplate;

    public List<User> getUsersByIds(List<Integer> ids) {
        User[] users = restTemplate.getForObject("http://hystrix-client-provider/user/{1}", User[].class, StringUtils.join(ids, ","));
        return Arrays.asList(users);
    }
}
```

新增 `UserCollapser` ，如下：

```java
public class UserCollapser extends HystrixCollapser<List<User>, User, Integer> {
    private UserService userService;
    private Integer id;

    public UserCollapser(UserService userService, Integer id) {
        super(Setter.withCollapserKey(HystrixCollapserKey.Factory.asKey("userCollapserKey")).andCollapserPropertiesDefaults(HystrixCollapserProperties.Setter().withTimerDelayInMilliseconds(200)));
        this.userService = userService;
        this.id = id;
    }

    /**
     * 请求参数
     *
     * @return
     */
    @Override
    public Integer getRequestArgument() {
        return id;
    }

    /**
     * 请求合并的方法
     *
     * @param collection
     * @return
     */
    @Override
    protected HystrixCommand<List<User>> createCommand(Collection<CollapsedRequest<User, Integer>> collection) {
        List<Integer> ids = new ArrayList<>(collection.size());
        for (CollapsedRequest<User, Integer> userIntegerCollapsedRequest : collection) {
            ids.add(userIntegerCollapsedRequest.getArgument());
        }
        return new UserCommand(ids, userService);
    }

    /**
     * 请求结果分发
     *
     * @param users
     * @param collection
     */
    @Override
    protected void mapResponseToRequests(List<User> users, Collection<CollapsedRequest<User, Integer>> collection) {
        int count = 0;
        for (CollapsedRequest<User, Integer> request : collection) {
            request.setResponse(users.get(count++));
        }
    }
}
```

新增 `UserCommand` ，如下：

```java
public class UserCommand extends HystrixCommand<List<User>> {
    private List<Integer> ids;
    private UserService userService;

    public UserCommand(List<Integer> ids, UserService userService) {
        super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("userCommandGroupKey")).andCommandKey(HystrixCommandKey.Factory.asKey("userCommandKey")));
        this.ids = ids;
        this.userService = userService;
    }

    @Override
    protected List<User> run() throws Exception {
        return userService.getUsersByIds(ids);
    }

    @Override
    protected List<User> getFallback() {
        return null;
    }
}
```

---

修改测试接口 `ConsumerController2` ，增加方法，如下：

```java
@Autowired
UserService userService;

// 请求合并
@GetMapping("/testHystrixCollapser2")
public void testHystrixCollapser2() throws ExecutionException, InterruptedException {
    HystrixRequestContext ctx = HystrixRequestContext.initializeContext();

    // 这3个请求会一起发起
    UserCollapser collapser1 = new UserCollapser(userService, 99);
    UserCollapser collapser2 = new UserCollapser(userService, 98);
    UserCollapser collapser3 = new UserCollapser(userService, 97);
    Future<User> q1 = collapser1.queue();
    Future<User> q2 = collapser2.queue();
    Future<User> q3 = collapser3.queue();
    User u1 = q1.get();
    User u2 = q2.get();
    User u3 = q3.get();
    System.out.println(u1);
    System.out.println(u2);
    System.out.println(u3);

    Thread.sleep(2000);

    // 这个请求会单独发起
    UserCollapser collapser4 = new UserCollapser(userService, 96);
    Future<User> q4 = collapser4.queue();
    User u4 = q4.get();
    System.out.println(u4);

    ctx.close();
}
```

访问 [http://127.0.0.1:3002/testHystrixCache2](http://127.0.0.1:3002/testHystrixCache2) 完成测试。

---

- [Spring Cloud 教程合集](https://mp.weixin.qq.com/s/SBmcs2bxumhNz4kky1pl-A)（微信左下方**阅读全文**可直达）。
- Spring Cloud 教程合集示例代码：[https://github.com/cxy35/spring-cloud-samples](https://github.com/cxy35/spring-cloud-samples)
- 本文示例代码：[https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-hystrix](https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-hystrix)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)