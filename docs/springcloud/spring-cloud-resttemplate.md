本文整理在 Spring Cloud 中 RestTemplate 的使用说明，包括 `GET/POST/PUT/DELETE` 请求以及一些通用的请求执行方法 `exchange/execute` ，结合 `@LoadBalanced` 注解可以实现客户端负载均衡。
<!-- more -->

## 1 概述

Java 中的 HTTP 请求工具有 HttpURLConnection（ JDK 自带）/HttpClient/OkHttp 等，此外还有一个 `RestTemplate` ，由 Spring 提供（但大家之前可能没怎么用过），它与 Spring Boot 和 Spring Cloud 都无关。 RestTemplate 提供了常见的 REST 请求方法模板，例如 `GET/POST/PUT/DELETE` 请求以及一些通用的请求执行方法 `exchange/execute` ，结合 `@LoadBalanced` 注解可以实现客户端负载均衡。

RestTemplate 本身实现了 RestOperations 接口，而在 RestOperations 接口中，定义了常见的 RESTful 操作，这些操作在 RestTemplate 中都得到了很好的实现。 RestTemplate 一般在 consumer 中使用，用来调用 provider 中的接口。

## 2 使用说明

在 eureka-client-provider 中新增 `ProviderController2` 用于测试 RestTemplate ，如下：

```java
@Controller
public class ProviderController2 {
}
```

### 2.1 GET

首先，我们在 provider 中定义一个 GET 接口：

```java
@GetMapping("/user")
@ResponseBody
public User findUser(String username) {
    User user = new User();
    user.setId(12);
    user.setUsername(username);
    user.setPassword("123456");
    return user;
}
```

接下来，在 consumer 去访问这个接口，这个接口是一个 GET 请求，所以，访问方式就是调用 RestTemplate 中的 GET 请求。可以看到，在 RestTemplate 中，关于 GET 请求，一共有如下两大类方法：

![](https://oscimg.oschina.net/oscnet/up-7b4d673cfc278efe8f27418ef1d9c8f4da1.png)

这两大类方法实际上是重载的，唯一不同的就是返回值类型。 `getForObject` 返回的是一个对象，这个对象就是服务端返回的具体值。 `getForEntity` 返回的是一个 ResponseEntity ，这个 ResponseEntity 中除了服务端返回的具体数据外，还保留了 HTTP 响应头的数
据。

```java
@GetMapping("/testGet")
public void testGet() {
    User user = restTemplateLoadBalanced.getForObject("http://provider/user?username={1}", User.class, "cxy35");
    System.out.println(user);

    ResponseEntity<User> responseEntity = restTemplateLoadBalanced.getForEntity("http://provider/user?username={1}", User.class, "cxy35");
    user = responseEntity.getBody();
    System.out.println("body:" + user);
    HttpStatus statusCode = responseEntity.getStatusCode();
    System.out.println("HttpStatus:" + statusCode);
    int statusCodeValue = responseEntity.getStatusCodeValue();
    System.out.println("statusCodeValue:" + statusCodeValue);
    HttpHeaders headers = responseEntity.getHeaders();
    Set<String> keySet = headers.keySet();
    System.out.println("--------------header-----------");
    for (String s : keySet) {
        System.out.println(s + ":" + headers.get(s));
    }
}
```

启动 Eureka Server、provider 以及 consumer 之后，访问 [http://127.0.0.1:1115/testGet](http://127.0.0.1:1115/testGet) 完成测试。

验证之后发现， getForObject 直接拿到了服务的返回值， getForEntity 不仅仅拿到服务的返回值，还拿到 http 响应的状态码。 getForObject 和 getForEntity 分别有三个重载方法，两者的三个重载方法基本都是一致的。三个重载方法，其实代表了三种不同的传参方式，这里以 getForObject 为例，如下：

```java
@GetMapping("/testGet2")
public void testGet2() throws UnsupportedEncodingException {
    // 传参方式：字符串
    User user = restTemplateLoadBalanced.getForObject("http://provider/user?username={1}", User.class, "cxy35");
    System.out.println(user);

    // 传参方式：Map
    Map<String, Object> map = new HashMap<>();
    map.put("username", "zhangsan");
    user = restTemplateLoadBalanced.getForObject("http://provider/user?username={username}", User.class, map);
    System.out.println(user);

    // 传参方式：URI
    String url = "http://provider/user?username=" + URLEncoder.encode("张三", "UTF-8");
    URI uri = URI.create(url);
    user = restTemplateLoadBalanced.getForObject(uri, User.class);
    System.out.println(user);
}
```

访问 [http://127.0.0.1:1115/testGet2](http://127.0.0.1:1115/testGet2) 完成测试。

### 2.2 POST

首先，在 provider 中提供两个 POST 接口：

```java
@PostMapping("/user")
@ResponseBody
public User addUser(User user) {
    return user;
}

@PostMapping("/user2")
@ResponseBody
public User addUser2(@RequestBody User user) {
    return user;
}
```

两个方法代表了两种不同的传参方式。第一种方法是以 key/value 形式来传参，第二种方法是以 JSON 形式来传参。定义完成后，接下来，我们在 consumer 中调用这两个 POST 接口。

![](https://oscimg.oschina.net/oscnet/up-e253483acd4fd49fbb43852d88c6dceca67.png)

可以看到，这里的 post 和前面的 get 非常像，只是多出来了三个方法，就是 `postForLocation` ，另外两个 `postForObject` 和 `postForEntiy` 和前面 get 基本一致，所以这里我们主要来看 postForObject ，看完之后，我们再来看这个额外的 postForLocation 。

---

```java
@GetMapping("/testPost")
public void testPost() {
    // 传参方式：key/value
    MultiValueMap<String, Object> map = new LinkedMultiValueMap<>();
    map.add("username", "cxy35");
    map.add("password", "123");
    map.add("id", 99);
    User user = restTemplateLoadBalanced.postForObject("http://provider/user", map, User.class);
    System.out.println(user);

    // 传参方式：JSON
    user.setId(98);
    user = restTemplateLoadBalanced.postForObject("http://provider/user2", user, User.class);
    System.out.println(user);
}
```

访问 [http://127.0.0.1:1115/testPost](http://127.0.0.1:1115/testPost) 完成测试。

post 参数到底是 key/value 形式还是 JSON 形式，主要看第二个参数，如果第二个参数是 MultiValueMap ，则参数是以 key/value 形式来传递的，如果是一个普通对象，则参数是以 JSON 形式来传递的。最后再看一下 postForLocation 。有的时候，当我执行完一个 post 请求之后，立马要进行重定向，一个非常常见的场景就是注册，注册是一个 post 请求，注册完成之后，立马重定向到登录页面去登录。对于这种场景，我们就可以使用 postForLocation 。

首先，我们在 provider 上提供一个用户注册接口，也是 POST 类型：

```java
@PostMapping("/register")
public String register(User user) {
    return "redirect:http://provider/loginPage?username=" + user.getUsername();
}

@GetMapping("/loginPage")
@ResponseBody
public String loginPage(String username) {
    return "loginPage: " + username;
}
```

注意：**这里的 post 接口，响应一定要是 302 ，否则 postForLocation 无效。重定向的地址，一定要写成绝对路径，不要写相对路径，否则在 consumer 中调用时会出问题**。

接着在 consumer 中新增接口，调用上述 provider 中的接口。

```java
@GetMapping("/testPost2")
public void testPost2() {
    // postForLocation 实现重定向
    MultiValueMap<String, Object> map = new LinkedMultiValueMap<>();
    map.add("username", "cxy35");
    map.add("password", "123");
    map.add("id", 99);
    URI uri = restTemplateLoadBalanced.postForLocation("http://provider/register", map);
    System.out.println(uri);
    
    String s = restTemplateLoadBalanced.getForObject(uri, String.class);
    System.out.println(s);
}
```

这就是 postForLocation ，调用该方法返回的是一个 URI ，这个 URI 就是重定向的地址（里边也包含了重定向的参数），拿到 URI 之后，就可以直接发送新的请求了。

访问 [http://127.0.0.1:1115/testPost2](http://127.0.0.1:1115/testPost2) 完成测试。

### 2.3 PUT

PUT 请求比较简单，重载的方法也比较少。我们首先在 provider 中提供两个 PUT 接口：

```java
@PutMapping("/user")
@ResponseBody
public void updateUser(User user) {
    System.out.println("updateUser: " + user);
}

@PutMapping("/user2")
@ResponseBody
public void updateUser2(@RequestBody User user) {
    System.out.println("updateUser2: " + user);
}
```

注意， PUT 接口传参其实和 POST 很像，也接受两种类型的参数， key/value 形式以及 JSON 形式。在 consumer 中，我们来调用该接口：

```java
@GetMapping("/testPut")
public void testPut() {
    // 传参方式：key/value
    MultiValueMap<String, Object> map = new LinkedMultiValueMap<>();
    map.add("username", "cxy35");
    map.add("password", "123");
    map.add("id", 99);
    restTemplateLoadBalanced.put("http://provider/user", map);

    // 传参方式：JSON
    User user = new User();
    user.setId(98);
    user.setUsername("zhangsan");
    user.setPassword("456");
    restTemplateLoadBalanced.put("http://provider/user2", user);
}
```

访问 [http://127.0.0.1:1115/testPut](http://127.0.0.1:1115/testPut) 完成测试。

consumer 中的写法基本和 POST 类似，也是两种方式，可以传递两种不同类型的参数。

![](https://oscimg.oschina.net/oscnet/up-26e6c0d0596ddbd7fbcd4ee43319ad54b00.png)

### 2.4 DELETE

DELETE 也比较容易，我们有两种方式来传递参数， key/value 形式或者 PathVariable（参数放在路径中），首先我们在 provider 中定义两个 DELETE 方法：

```java
@DeleteMapping("/user")
@ResponseBody
public void deleteUser(Integer id) {
    System.out.println("deleteUser: " + id);
}

@DeleteMapping("/user2/{id}")
@ResponseBody
public void deleteUser2(@PathVariable Integer id) {
    System.out.println("deleteUser2: " + id);
}
```

然后在 consumer 中调用这两个删除的接口：

```java
@GetMapping("/testDelete")
public void testDelete() {
    restTemplateLoadBalanced.delete("http://provider/user?id={1}", 99);

    restTemplateLoadBalanced.delete("http://provider/user2/{1}", 99);
}
```

访问 [http://127.0.0.1:1115/testDelete](http://127.0.0.1:1115/testDelete) 完成测试。

delete 中参数的传递，也支持 map ，这块实际上和 get 是一样的。

![](https://oscimg.oschina.net/oscnet/up-3d587c4f590a7ad4789b50e8f495e255b5f.png)

## 3 客户端负载均衡

我们知道在 RestTemplate 上添加 `@LoadBalanced` 注解可以实现负载均衡功能，这里实现的其实是**客户端负载均衡**，客户端负载均衡是相对服务端负载均衡而言的。

平时我们用 Nginx 实现的叫**服务端负载均衡**。它的一个特点是，就是调用的客户端并不知道具体是哪一个 Server 提供的服务，它也不关心，反正请求发送给 Nginx ， Nginx 再将请求转发给 Tomcat ，客户端只需要记着 Nginx 的地址即可，如下：

![](https://oscimg.oschina.net/oscnet/up-b1394a07c2cd55f6450c2ae377caa5318ea.png)

客户端负载均衡则是另外一种情形，调用的客户端本身是知道所有 Server 的详细信息的，当需要调用 Server 上的接口的时候，客户端从自身所维护的 Server 列表中，根据提前配置好的负载均衡策略，自己挑选一个 Server 来调用，此时，客户端知道它所调用的是哪一个 Server ，如下：

![](https://oscimg.oschina.net/oscnet/up-edf8d3b88563b5dc553f137da6823314ab1.png)

在 Spring Cloud 中，要想使用负载均衡功能，只需要给 RestTemplate 实例上添加一个 @LoadBalanced 注解即可，此时， RestTemplate 就会自动具备负载均衡功能，这个负载均衡就是客户端负载均衡。

上述功能的实现主要分为三步：

1. 服务消费方/调用方（ consumer ）从 Eureka Client 本地缓存的服务注册信息中，选择一个可以调用的服务（ provider ）。
2. 根据 1 中所选择的服务，重构请求 URL 地址。
3. 将 1、2 步的功能嵌入到 RestTemplate 中。

---

- [Spring Cloud 教程合集](https://mp.weixin.qq.com/s/SBmcs2bxumhNz4kky1pl-A)（微信左下方**阅读全文**可直达）。
- Spring Cloud 教程合集示例代码：[https://github.com/cxy35/spring-cloud-samples](https://github.com/cxy35/spring-cloud-samples)
- 本文示例代码：[https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-netflix-eureka/eureka-client-consumer](https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-netflix-eureka/eureka-client-consumer)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)