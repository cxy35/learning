---
title: Spring Boot 整合 Spring Security（配置登录/登出）
date: 2020-01-18 19:37:17
categories: Spring Boot
tags: [Spring Boot, Spring Security]
toc: true
---
Spring Boot 整合 Spring Security ，配置登录/登出，如：登录接口，登录成功或失败后的响应等。
<!-- more -->

## 1 创建工程

创建 Spring Boot 项目 `spring-boot-springsecurity-login` ，添加 `Web/Spring Security` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-1d23871e606c43a843a073470a40bc2081d.png)

最终的依赖如下：

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

## 2 配置 Spring Security

新增 `SecurityConfig` 配置类，如下：

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    PasswordEncoder passwordEncoder() {
//        return NoOpPasswordEncoder.getInstance();// 密码不加密
        return new BCryptPasswordEncoder();// 密码加密
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // 在内存中配置 2 个用户
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
        // 开启登录配置
        http.authorizeRequests()
                // 表示 admin 角色能访问
                .antMatchers("/admin/**").hasRole("admin")
                // 表示 admin 或 user 角色都能访问
                // .antMatchers("/user/**").hasAnyRole("admin", "user")
                // 表示 admin 或 user 角色都能访问
                .antMatchers("/user/**").access("hasAnyRole('admin','user')")
                // 表示剩余的其他接口，登录之后就能访问
                .anyRequest().authenticated()
                .and()
                .formLogin()
                // 表示登录页的地址，例如当你访问一个需要登录后才能访问的资源时，系统就会自动给你通过【重定向】跳转到这个页面上来
                .loginPage("/login")
                // 表示处理登录请求的接口地址，默认为 /login
                .loginProcessingUrl("/doLogin")
                // 定义登录时，用户名的 key，默认为 username
                .usernameParameter("uname")
                // 定义登录时，密码的 key，默认为 password
                .passwordParameter("passwd")
                // 登录成功的处理器
                .successHandler(new AuthenticationSuccessHandler() {
                    @Override
                    public void onAuthenticationSuccess(HttpServletRequest req, HttpServletResponse resp, Authentication authentication) throws IOException, ServletException {
                        resp.setContentType("application/json;charset=utf-8");
                        PrintWriter out = resp.getWriter();
                        Map<String, Object> map = new HashMap<>();
                        map.put("status", 200);
                        map.put("msg", authentication.getPrincipal());
                        out.write(new ObjectMapper().writeValueAsString(map));
                        out.flush();
                        out.close();
                    }
                })
                // 登录失败的处理器
                .failureHandler(new AuthenticationFailureHandler() {
                    @Override
                    public void onAuthenticationFailure(HttpServletRequest req, HttpServletResponse resp, AuthenticationException e) throws IOException, ServletException {
                        resp.setContentType("application/json;charset=utf-8");
                        PrintWriter out = resp.getWriter();
                        Map<String, Object> map = new HashMap<>();
                        map.put("status", 401);
                        if (e instanceof LockedException) {
                            map.put("msg", "账户被锁定，登录失败!");
                        } else if (e instanceof BadCredentialsException) {
                            map.put("msg", "用户名或密码输入错误，登录失败!");
                        } else if (e instanceof DisabledException) {
                            map.put("msg", "账户被禁用，登录失败!");
                        } else if (e instanceof AccountExpiredException) {
                            map.put("msg", "账户过期，登录失败!");
                        } else if (e instanceof CredentialsExpiredException) {
                            map.put("msg", "密码过期，登录失败!");
                        } else {
                            map.put("msg", "登录失败!");
                        }
                        out.write(new ObjectMapper().writeValueAsString(map));
                        out.flush();
                        out.close();
                    }
                })
                // 和表单登录相关的接口统统都直接通过
                .permitAll()
                .and()
                .logout()
                .logoutUrl("/logout")
                // 登出成功的处理器
                .logoutSuccessHandler(new LogoutSuccessHandler() {
                    @Override
                    public void onLogoutSuccess(HttpServletRequest req, HttpServletResponse resp, Authentication authentication) throws IOException, ServletException {
                        resp.setContentType("application/json;charset=utf-8");
                        PrintWriter out = resp.getWriter();
                        Map<String, Object> map = new HashMap<>();
                        map.put("status", 200);
                        map.put("msg", "注销登录成功!");
                        out.write(new ObjectMapper().writeValueAsString(map));
                        out.flush();
                        out.close();
                    }
                })
                .permitAll()
                .and()
                .csrf().disable()
                .exceptionHandling()
                // 无访问权限的处理器
                .accessDeniedHandler(new AccessDeniedHandler() {
                    @Override
                    public void handle(HttpServletRequest req, HttpServletResponse resp, AccessDeniedException e) throws IOException, ServletException {
                        resp.setContentType("application/json;charset=utf-8");
                        PrintWriter out = resp.getWriter();
                        Map<String, Object> map = new HashMap<>();
                        map.put("status", 403);
                        map.put("msg", "无访问权限!");
                        out.write(new ObjectMapper().writeValueAsString(map));
                        out.flush();
                        out.close();
                    }
                })
                // 默认情况下用户直接访问一个需要认证之后才可以访问的请求时，会被重定向到.loginPage("/login")，前后端分离时会导致跨域。
                // 增加如下配置后，就不会发生重定向操作了，服务端会直接给浏览器一个 JSON 提示
                .authenticationEntryPoint(new AuthenticationEntryPoint() {
                    @Override
                    public void commence(HttpServletRequest req, HttpServletResponse resp, AuthenticationException authException) throws IOException, ServletException {
                        resp.setContentType("application/json;charset=utf-8");
                        PrintWriter out = resp.getWriter();
                        Map<String, Object> map = new HashMap<>();
                        map.put("status", 401);
                        if (authException instanceof InsufficientAuthenticationException) {
                            map.put("msg", "访问失败，请先登录!");
                        } else {
                            map.put("msg", "访问失败!");
                        }
                        out.write(new ObjectMapper().writeValueAsString(map));
                        out.flush();
                        out.close();
                    }
                });
    }

    @Override
    public void configure(WebSecurity web) throws Exception {
        // 配置不需要拦截的请求地址，即该地址不走 Spring Security 过滤器链
        web.ignoring().antMatchers("/vercode");
    }
}
```

## 3 测试

新增 `HelloController` 测试类，如下：

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

    @GetMapping("/login")
    public String login() {
        return "please login";
    }
}
```

项目启动之后，用 `Postman` 完成测试，如下：

访问 `/hello` 接口，提示先登录。

![](https://oscimg.oschina.net/oscnet/up-4c1e17f382bb7eb676f12c4416694ad6c29.png)

访问 `/doLogin` 接口登录失败，因为 `key` 不对。

![](https://oscimg.oschina.net/oscnet/up-9bc44ec6c06b38e3700cfde7bf1019ae75d.png)

用自定义的 `key` 访问 `/doLogin` 接口登录成功。

![](https://oscimg.oschina.net/oscnet/up-b342180e83e7d9f137bde8eedda14e8150f.png)

再访问 `/hello` 接口，返回正常。

![](https://oscimg.oschina.net/oscnet/up-5478851d9cbc3b56971dd8f7219e2ae46ac.png)

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-security/spring-boot-springsecurity-login](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-security/spring-boot-springsecurity-login)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)