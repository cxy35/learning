---
title: Spring Boot 整合 Spring Security + JWT（实现无状态登录）
date: 2020-01-25 16:26:52
categories: Spring Boot
tags: [Spring Boot, Spring Security, JWT]
toc: true
---
学习在 Spring Boot 中整合 Spring Security 和 JWT ，实现无状态登录，可做为前后端分离时的解决方案，技术上没问题，但实际上还是推荐使用 OAuth2 中的 password 模式。
<!-- more -->

## 1 登录概述

### 1.1 有状态登录

有状态服务，即服务端需要记录每次会话的客户端信息，从而识别客户端身份，根据用户身份进行请求的处理，如 Tomcat 中的 Session 。例如：用户登录后，我们把用户的信息保存在服务端 session 中，并且给用户一个 cookie 值，记录对应的 session ，然后下次请求，用户携带 cookie 值来（这一步由浏览器自动完成），我们就能识别到对应 session ，从而找到用户的信息。这种方式目前来看最方便，但是也有一些缺陷，如下：

- 服务端保存大量数据，增加服务端压力。
- 服务端保存用户状态，不支持集群化部署。

### 1.2 无状态登录

微服务集群中的每个服务，对外提供的都使用 RESTful 风格的接口。而 RESTful 风格的一个最重要的规范就是：服务的无状态性，即：

- 服务端不保存任何客户端请求者信息。
- 客户端的每次请求必须具备自描述信息，通过这些信息识别客户端身份。

优势：

- 客户端请求不依赖服务端的信息，多次请求不需要必须访问到同一台服务器。
- 服务端的集群和状态对客户端透明。
- 服务端可以任意的迁移和伸缩（可以方便的进行集群化部署）。
- 减小服务端存储压力。

### 1.3 无状态登录的流程

无状态登录的流程：

1. 首先客户端发送账户名/密码到服务端进行认证。
2. 认证通过后，服务端将用户信息加密并且编码成一个 token ，返回给客户端。
3. 以后客户端每次发送请求，都需要携带认证的 token 。
4. 服务端对客户端发送来的 token 进行解密，判断是否有效，并且获取用户登录信息。

## 2 JWT 概述

### 2.1 JWT 简介

JWT (Json Web Token)，是一种 JSON 风格的轻量级的授权和身份认证规范，可实现无状态、分布式的 Web 应用授权。官网：[https://jwt.io/](https://jwt.io/)

![](https://oscimg.oschina.net/oscnet/up-d974d409831766a8f03a26514bfae9da5e3.png)

JWT 作为一种规范，并没有和某一种语言绑定在一起，常用的 Java 实现是 GitHub 上的开源项目 **`jjwt`** ，地址如下：[https://github.com/jwtk/jjwt](https://github.com/jwtk/jjwt)

### 2.2 JWT 数据格式

JWT 包含三部分数据：

1. **`Header`** ：头部，通常头部有两部分信息：

    - 声明类型，这里是 JWT 。
    - 加密算法，自定义。

    我们会对头部进行 Base64 编码（可解码），得到第一部分数据。

2. **`Payload`** ：载荷，就是有效数据，在官方文档中(RFC7519)，这里给了 7 个示例信息：

    - iss (issuer)：表示签发人。
    - exp (expiration time)：表示token过期时间。
    - sub (subject)：主题。
    - aud (audience)：受众。
    - nbf (Not Before)：生效时间。
    - iat (Issued At)：签发时间。
    - jti (JWT ID)：编号。

    这部分也会采用 Base64 编码，得到第二部分数据。

3. **`Signature`** ：签名，是整个数据的认证信息。一般根据前两步的数据，再加上服务端的密钥 secret （密钥保存在服务端，不能泄露给客户端），通过 Header 中配置的加密算法生成，用于验证整个数据的完整性和可靠性。

比如，生成的数据格式：

`eyJhbGciOiJIUzUxMiJ9.eyJhdXRob3JpdGllcyI6IlJPTEVfdXNlciwiLCJzdWIiOiJ1c2VyIiwiZXhwIjoxNTc0NzczNTkyfQ.FuPIltzXi5j14t_gSL1GoIMUZxTHKK0FvB3gds6eTZFDkQr1ZxWVxdqZ5YFbCxdkwQ_VXtPK-GgcW5Kzzx3wvw`

注意，这里的数据通过 `.` 隔开成了三部分，分别对应前面提到的三部分：

1. Header ：头部（声明类型、加密算法），采用 Base64 编码，如：`eyJhbGciOiJIUzUxMiJ9` 。
2. Payload ：载荷，就是有效数据，采用 Base64 编码，如：`eyJhdXRob3JpdGllcyI6IlJPTEVfdXNlciwiLCJzdWIiOiJ1c2VyIiwiZXhwIjoxNTc0NzczNTkyfQ`
3. Signature ：签名，如：`FuPIltzXi5j14t_gSL1GoIMUZxTHKK0FvB3gds6eTZFDkQr1ZxWVxdqZ5YFbCxdkwQ_VXtPK-GgcW5Kzzx3wvw` 。

### 2.3 JWT 交互流程

![](https://oscimg.oschina.net/oscnet/up-0998cf2a8b784c5bb10aa2fcc4151e99221.png)

1. 应用程序或客户端向授权服务器请求授权。
2. 获取到授权后，授权服务器会向应用程序返回访问令牌。
3. 应用程序使用访问令牌来访问受保护资源（如 API ）。

因为 JWT 签发的 token 中已经包含了用户的身份信息，并且每次请求都会携带，这样服务端就无需保存用户信息，甚至无需去数据库查询，这样就完全符合了 RESTful 的无状态规范。

### 2.4 JWT 问题

说了这么多， JWT 也不是天衣无缝，由客户端维护登录状态带来的一些问题在这里依然存在，如下：

1. 续签问题，这是被很多人诟病的问题之一，传统的 cookie + session 的方案天然的支持续签，但是 JWT 由于服务端不保存用户状态，因此很难完美解决续签问题，如果引入 Redis ，虽然可以解决问题，但是 JWT 也变得不伦不类了。
2. 注销问题，由于服务端不再保存用户信息，所以一般可以通过修改 secret 来实现注销，服务端 secret 修改后，已经颁发的未过期的 token 就会认证失败，进而实现注销，不过毕竟没有传统的注销方便。
3. 密码重置，密码重置后，原本的 token 依然可以访问系统，这时候也需要强制修改 secret 。
4. 基于第 2 点和第 3 点，一般建议不同用户取不同 secret 。

## 3 实战

### 3.1 创建工程

创建 Spring Boot 项目 `spring-boot-springsecurity-jwt` ，添加 `Web/Spring Security` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-1d23871e606c43a843a073470a40bc2081d.png)

之后手动在 pom 文件中添加 `jjwt` 依赖，最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt</artifactId>
        <version>0.9.1</version>
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
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 3.2 创建接口

新建实体类 `User` 实现 `UserDetails` 接口，如下：

```java
public class User implements UserDetails {
    private String username;
    private String password;
    private List<GrantedAuthority> authorities;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return authorities;
    }

    @Override
    public String getPassword() {
        return password;
    }

    @Override
    public String getUsername() {
        return username;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public void setAuthorities(List<GrantedAuthority> authorities) {
        this.authorities = authorities;
    }
}
```

---

新建 `HelloController` ，如下：

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }

    @GetMapping("/admin/hello")
    public String admin() {
        return "hello admin";
    }

    @GetMapping("/user/hello")
    public String user() {
        return "hello user";
    }
}
```

### 3.3 配置过滤器

这里主要配置两个过滤器：

- 用户登录的过滤器

```java
// 过滤器1：用户登录的过滤器，在用户的登录的过滤器中校验用户是否登录成功，
// 如果登录成功，则生成一个 token 返回给客户端，登录失败则给前端一个登录失败的提示
public class JwtLoginFilter extends AbstractAuthenticationProcessingFilter {

    public JwtLoginFilter(String defaultFilterProcessesUrl, AuthenticationManager authenticationManager) {
        super(new AntPathRequestMatcher(defaultFilterProcessesUrl));
        setAuthenticationManager(authenticationManager);
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest req, HttpServletResponse httpServletResponse) throws AuthenticationException, IOException {
        // 这里只支持 JSON 的登录方式
        // 如果想表单方式也支持，可参考 spring-boot-springsecurity-loginbyjson 中的 MyAuthenticationFilter
        // 获取输入参数，如 {"username":"user","password":"123456"}
        User user = new ObjectMapper().readValue(req.getInputStream(), User.class);
        // 进行登录校验，如果校验成功，会到 successfulAuthentication 的回调中，否则到 unsuccessfulAuthentication 的回调中
        return getAuthenticationManager().authenticate(new UsernamePasswordAuthenticationToken(user.getUsername(), user.getPassword()));
    }

    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse resp, FilterChain chain, Authentication authResult) throws IOException, ServletException {
        // 获取登录用户的角色
        Collection<? extends GrantedAuthority> authorities = authResult.getAuthorities();
        StringBuffer sb = new StringBuffer();
        for (GrantedAuthority authority : authorities) {
            sb.append(authority.getAuthority()).append(",");
        }

        // 生成 token 并返回
        // 数据格式：分 3 部分用 . 隔开，如：eyJhbGciOiJIUzUxMiJ9.eyJhdXRob3JpdGllcyI6IlJPTEVfdXNlciwiLCJzdWIiOiJ1c2VyIiwiZXhwIjoxNTc0NzczNTkyfQ.FuPIltzXi5j14t_gSL1GoIMUZxTHKK0FvB3gds6eTZFDkQr1ZxWVxdqZ5YFbCxdkwQ_VXtPK-GgcW5Kzzx3wvw
        // 1.Header：头部（声明类型、加密算法），采用 Base64 编码，如：eyJhbGciOiJIUzUxMiJ9
        // 2.Payload：载荷，就是有效数据，采用 Base64 编码，如：eyJhdXRob3JpdGllcyI6IlJPTEVfdXNlciwiLCJzdWIiOiJ1c2VyIiwiZXhwIjoxNTc0NzczNTkyfQ
        // 3.Signature：签名，是整个数据的认证信息。一般根据前两步的数据，再加上服务的的密钥 secret （密钥保存在服务端，不能泄露给客户端），通过 Header 中配置的加密算法生成。用于验证整个数据完整和可靠性。
        String jwt = Jwts.builder()
                .claim("authorities", sb) // 配置用户角色
                .setSubject(authResult.getName()) // 配置主题
                .setExpiration(new Date(System.currentTimeMillis() + 60 * 60 * 1000)) // 配置过期时间
                .signWith(SignatureAlgorithm.HS512, "abc@123") // 配置加密算法和密钥
                .compact();
        resp.setContentType("application/json;charset=utf-8");
        Map<String, String> map = new HashMap<>();
        map.put("token", jwt);
        map.put("msg", "登录成功");
        PrintWriter out = resp.getWriter();
        out.write(new ObjectMapper().writeValueAsString(map));
        out.flush();
        out.close();
    }

    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest req, HttpServletResponse resp, AuthenticationException failed) throws IOException, ServletException {
        resp.setContentType("application/json;charset=utf-8");
        Map<String, String> map = new HashMap<>();
        map.put("msg", "登录失败");
        PrintWriter out = resp.getWriter();
        out.write(new ObjectMapper().writeValueAsString(map));
        out.flush();
        out.close();
    }
}
```

---

- 校验 token 的过滤器

```java
// 过滤器2：当其他请求发送来，校验 token 的过滤器，如果校验成功，就让请求继续执行
// 请求时注意认证方式选择 Bearer Token
public class JwtFilter extends GenericFilterBean {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) servletRequest;
        // 获取 token ，注意获取方式要跟前台传的方式保持一致
        // 这里请求时注意认证方式选择 Bearer Token，会用 header 传递
        String jwtToken = req.getHeader("authorization");
        // 注意 "abc@123" 要与生成 token 时的保持一致
        Jws<Claims> jws = Jwts.parser().setSigningKey("abc@123")
                .parseClaimsJws(jwtToken.replace("Bearer", ""));
        Claims claims = jws.getBody();
        // 获取用户名
        String username = claims.getSubject();
        // 获取用户角色，注意 "authorities" 要与生成 token 时的保持一致
        List<GrantedAuthority> authorities = AuthorityUtils.commaSeparatedStringToAuthorityList((String) claims.get("authorities"));
        UsernamePasswordAuthenticationToken token = new UsernamePasswordAuthenticationToken(username, null, authorities);
        SecurityContextHolder.getContext().setAuthentication(token);
        filterChain.doFilter(servletRequest, servletResponse);
    }
}
```

### 3.4 配置 Spring Security

新增 `SecurityConfig` 配置类，如下：

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    PasswordEncoder passwordEncoder() {
        // return NoOpPasswordEncoder.getInstance();// 密码不加密
        return new BCryptPasswordEncoder();// 密码加密
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // 在内存中配置2个用户
        /*auth.inMemoryAuthentication()
                .withUser("admin").password("123456").roles("admin")
                .and()
                .withUser("user").password("123456").roles("user");// 密码不加密*/

        auth.inMemoryAuthentication()
                .withUser("admin").password("$2a$10$fB2UU8iJmXsjpdk6T6hGMup8uNcJnOGwo2.QGR.e3qjIsdPYaS4LO").roles("admin")
                .and()
                .withUser("user").password("$2a$10$3TQ2HO/Xz1bVHw5nlfYTBON2TDJsQ0FMDwAS81uh7D.i9ax5DR46q").roles("user");// 密码加密
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/admin/**").hasRole("admin")
                .antMatchers("/user/**").access("hasAnyRole('user','admin')")
                .antMatchers(HttpMethod.POST, "/login").permitAll()
                .anyRequest().authenticated()
                .and()
                .addFilterBefore(new JwtLoginFilter("/login", authenticationManager()), UsernamePasswordAuthenticationFilter.class)
                .addFilterBefore(new JwtFilter(), UsernamePasswordAuthenticationFilter.class)
                .csrf().disable();
    }
}
```

### 3.5 测试

项目启动之后，用 `Postman` 完成测试。

![](https://oscimg.oschina.net/oscnet/up-762d920371a8e533a0db695bd6f57110c41.png)

取出 token 的第 1 部分， Base64 解码得到 Header ，如下：

![](https://oscimg.oschina.net/oscnet/up-e2ac16a021a6a82e68501f5902800454949.png)

取出 token 的第 2 部分， Base64 解码得到 Payload ，如下：

![](https://oscimg.oschina.net/oscnet/up-a994fbd93cdd3256be5244407dca22ce560.png)

因为 Base64 是一种编码方案，并不是加密方案，因此不建议将用户的敏感信息放在 token 中。

---

最后拿着上述 token 访问 `/user/hello` ，可正常访问。注意：认证方式 Authorization 选择 Bearer Token 。

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-security/spring-boot-springsecurity-jwt](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-security/spring-boot-springsecurity-jwt)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)