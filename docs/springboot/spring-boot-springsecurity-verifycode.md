---
title: Spring Boot 整合 Spring Security（配置验证码）
date: 2020-01-16 18:04:27
categories: Spring Boot
tags: [Spring Boot, Spring Security]
toc: true
---
Spring Boot 整合 Spring Security ，配置验证码。
<!-- more -->

## 1 创建工程

创建 Spring Boot 项目 `spring-boot-springsecurity-verifycode` ，添加 `Web/Spring Security` 依赖，如下：

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

## 2 准备验证码和接口

新增 `VerifyCode` 验证码生成工具类，如下：

```java
// 生成验证码的工具类
public class VerifyCode {

    private int width = 100;// 生成验证码图片的宽度
    private int height = 50;// 生成验证码图片的高度
    private String[] fontNames = {"宋体", "楷体", "隶书", "微软雅黑"};
    private Color bgColor = new Color(255, 255, 255);// 定义验证码图片的背景颜色为白色
    private Random random = new Random();
    private String codes = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
    private String text;// 记录随机字符串

    /**
     * 获取一个随意颜色
     *
     * @return
     */
    private Color randomColor() {
        int red = random.nextInt(150);
        int green = random.nextInt(150);
        int blue = random.nextInt(150);
        return new Color(red, green, blue);
    }

    /**
     * 获取一个随机字体
     *
     * @return
     */
    private Font randomFont() {
        String name = fontNames[random.nextInt(fontNames.length)];
        int style = random.nextInt(4);
        int size = random.nextInt(5) + 24;
        return new Font(name, style, size);
    }

    /**
     * 获取一个随机字符
     *
     * @return
     */
    private char randomChar() {
        return codes.charAt(random.nextInt(codes.length()));
    }

    /**
     * 创建一个空白的BufferedImage对象
     *
     * @return
     */
    private BufferedImage createImage() {
        BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
        Graphics2D g2 = (Graphics2D) image.getGraphics();
        g2.setColor(bgColor);// 设置验证码图片的背景颜色
        g2.fillRect(0, 0, width, height);
        return image;
    }

    public BufferedImage getImage() {
        BufferedImage image = createImage();
        Graphics2D g2 = (Graphics2D) image.getGraphics();
        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < 4; i++) {
            String s = randomChar() + "";
            sb.append(s);
            g2.setColor(randomColor());
            g2.setFont(randomFont());
            float x = i * width * 1.0f / 4;
            g2.drawString(s, x, height - 15);
        }
        this.text = sb.toString();
        drawLine(image);
        return image;
    }

    /**
     * 绘制干扰线
     *
     * @param image
     */
    private void drawLine(BufferedImage image) {
        Graphics2D g2 = (Graphics2D) image.getGraphics();
        int num = 5;
        for (int i = 0; i < num; i++) {
            int x1 = random.nextInt(width);
            int y1 = random.nextInt(height);
            int x2 = random.nextInt(width);
            int y2 = random.nextInt(height);
            g2.setColor(randomColor());
            g2.setStroke(new BasicStroke(1.5f));
            g2.drawLine(x1, y1, x2, y2);
        }
    }

    public String getText() {
        return text;
    }

    public static void output(BufferedImage image, OutputStream out) throws IOException {
        ImageIO.write(image, "JPEG", out);
    }
}
```

---

新增 `VerifyCodeController` ，提供验证码的获取接口，如下：

```java
@RestController
public class VerifyCodeController {
    @GetMapping("/vercode")
    public void code(HttpServletRequest req, HttpServletResponse resp, Model model) throws IOException {
        VerifyCode vc = new VerifyCode();
        BufferedImage image = vc.getImage();
        String text = vc.getText();
        model.addAttribute("text", text);
        System.out.println("text = " + text);
        HttpSession session = req.getSession();
        session.setAttribute("s_vercode", text);
        VerifyCode.output(image, resp.getOutputStream());
    }
}
```

---

在 `resources/static` 下新增 `vercode.html` 展示验证码，如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>测试验证码</title>
</head>
<body>
  <img src="/vercode" alt="">
</body>
</html>
```

访问 [http://127.0.0.1:8080/vercode.html](http://127.0.0.1:8080/vercode.html) ，效果如下：

![](https://oscimg.oschina.net/oscnet/up-599b568f6fb6a1c84453d7012bd3a976f36.png)

## 3 配置验证码过滤器

新增 `VerifyCodeFilter` 验证码过滤器

```java
// 验证码过滤器
@Component
public class VerifyCodeFilter extends GenericFilterBean {
    private String defaultFilterProcessUrl = "/doLogin";

    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException, AuthenticationServiceException {
        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) res;
        if ("POST".equalsIgnoreCase(request.getMethod()) && defaultFilterProcessUrl.equals(request.getServletPath())) {
            // 验证码验证
            String requestCaptcha = request.getParameter("vercode");
            String genCaptcha = (String) request.getSession().getAttribute("s_vercode");
            if (StringUtils.isEmpty(requestCaptcha) || StringUtils.isEmpty(genCaptcha))
                throw new AuthenticationServiceException("验证码不能为空!");
            if (!genCaptcha.toLowerCase().equals(requestCaptcha.toLowerCase())) {
                throw new AuthenticationServiceException("验证码错误!");
            }
        }
        chain.doFilter(request, response);
    }
}
```

当请求方法是 `POST` ，并且请求地址是 `/doLogin` 时，获取参数中的 `vercode` 字段值，该字段保存了用户从前端页面传来的验证码，然后获取 `session` 中保存的验证码，如果用户没有传来验证码，则抛出“验证码不能为空”异常，如果用户传入了验证码，则判断验证码是否正确，如果不正确则抛出“验证码错误”异常，否则执行 `chain.doFilter(request, response);` 使请求继续向下走。

## 4 配置 Spring Security

新增 `SecurityConfig` 配置类，如下：

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    VerifyCodeFilter verifyCodeFilter; // 验证码过滤器

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
        http.addFilterBefore(verifyCodeFilter, UsernamePasswordAuthenticationFilter.class);
        http.authorizeRequests()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .loginProcessingUrl("/doLogin")
                // 登录失败的处理器（VerifyCodeFilter 抛出的异常不会到这里？）
                .failureHandler(new AuthenticationFailureHandler() {
                    @Override
                    public void onAuthenticationFailure(HttpServletRequest req, HttpServletResponse resp, AuthenticationException e) throws IOException, ServletException {
                        resp.setContentType("application/json;charset=utf-8");
                        PrintWriter out = resp.getWriter();
                        Map<String, Object> map = new HashMap<>();
                        map.put("status", 401);
                        map.put("msg", e.getMessage());
                        out.write(new ObjectMapper().writeValueAsString(map));
                        out.flush();
                        out.close();
                    }
                })
                .permitAll()
                .and()
                .csrf().disable();
                // 其他配置可参考 spring-boot-springsecurity-login
    }

    @Override
    public void configure(WebSecurity web) throws Exception {
        // 配置不需要拦截的请求地址，即该地址不走 Spring Security 过滤器链
        web.ignoring().antMatchers("/vercode");
        web.ignoring().antMatchers("/vercode.html");
    }
}
```

配置之后，在登录时就要求传验证码了，如果不传或传错，控制台就会抛出异常。

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-security/spring-boot-springsecurity-verifycode](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-security/spring-boot-springsecurity-verifycode)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)