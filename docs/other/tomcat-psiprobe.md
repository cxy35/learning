---
title: Tomcat 管理和监控工具 - PSI Probe
date: 2019-05-24 10:50:50
categories: Tomcat
tags: [Tomcat, 工具, PSI Probe]
toc: true
---
使用 PSI Probe 管理和监控 Tomcat 。
<!-- more -->

## 1 准备工作

[PSI Probe](https://www.oschina.net/p/psiprobe) 是一个 Lambda Probe 的后续版本，主要是为了替换 Tomcat 自带的管理器，可方便的管理和监控 Tomcat 实例。主要特性：

- **Requests:** Monitor traffic in real-time, even on a per-application basis.
- **Sessions:** Browse/search attributes, view last IP, expire, estimate size.
- **JSP:** Browse, view source, compile.
- **Data Sources:** View pool usage, execute queries.
- **Logs:** View contents, download, change levels at runtime.
- **Threads:** View execution stack, kill.
- **Connectors:** Status, usage charts.
- **Cluster:** Status, usage charts.
- **JVM:** Memory usage charts, advise GC
- **Java Service Wrapper:** Restart JVM.
- **System:** CPU usage, memory usage, swap file usage.

---

- Tomcat 版本：apache-tomcat-8.0.53-windows-x64
- JDK 版本：jdk-8u181-windows-x64
- PSI Probe 版本：psi-probe-3.2.0

---

- 下载 probe.war：[https://github.com/psi-probe/psi-probe/releases](https://github.com/psi-probe/psi-probe/releases)

安装并启动成功。

## 2 配置 Tomcat

将打包或下载的 probe.war 放在 Tomcat 根目录 /webapps 目录下。

编辑 Tomcat 根目录下 /webapps/manager/META-INF/**context.xml** ，修改配置：

```xml
<!-- 注释掉Valve，如果不想注释，allow可增加需要访问的ip地址即可 -->
<!--
  <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
-->
```

## 3 测试

通过 [http://127.0.0.1:8080/probe/index.htm](http://127.0.0.1:8080/probe/index.htm) 访问，手动输入 tomcat-users.xml 中的用户名和密码登录后，查看页面截图如下：

![](https://oscimg.oschina.net/oscnet/7645a4e3608a559d4d2145856a4fb081a9f.jpg)

## 4 二次开发

**基于 psi-probe-3.2.0 二次开发，主要实现自动登录功能，使用 tomcat-users.xml 中的用户名和密码。**

涉及 2 个子工程修改：psi-probe-web 和 psi-probe-core 。工程代码：[https://github.com/cxy35/psi-probe](https://github.com/cxy35/psi-probe)

### 4.1 编辑 web.xml

编辑 psi-probe-web 工程目录下 /WEB-INF/web.xml ，修改如下：

```xml
<!-- 注释掉原来的 login-config 配置 -->
<!-- 
<login-config>
	<auth-method>BASIC</auth-method>
	<realm-name>PSI Probe</realm-name>
</login-config>
-->
<!-- 增加自己的 login-config 配置 -->
<!-- 配置认证方式为 form , 指定登陆页面 login.jsp 和登陆失败页面 loginError.jsp 。用户请求受保护资源时，会被容器踢回到 login.jsp ，获取用户名和密码自动提交给 j_security_check ，容器进行安全认证，方式是用户指定好的，认证成功后，马上进入到用户输入的受保护资源 -->
<login-config>
    <auth-method>FORM</auth-method>
    <form-login-config>
        <form-login-page>/myjsp/login.jsp</form-login-page>
        <form-error-page>/myjsp/loginError.jsp</form-error-page>
    </form-login-config>
</login-config>
```

### 4.2 编辑 decorators.xml

编辑 psi-probe-web 工程目录下 /WEB-INF/decorators.xml ，修改如下：

```xml
<excludes>
	<pattern>/*.xml.htm</pattern>
	<pattern>/*.ajax*</pattern>
	<pattern>/WEB-INF/*</pattern>
    <!-- 增加如下配置 -->
	<pattern>/myjsp/*</pattern>
</excludes>
```

### 4.3 新增 login.jsp

在 psi-probe-web 工程目录下增加 /myjsp/login.jsp ，内容如下：

```html
<%--

    Licensed under the GPL License. You may not use this file except in compliance with the License.
    You may obtain a copy of the License at

      https://www.gnu.org/licenses/old-licenses/gpl-2.0.html

    THIS PACKAGE IS PROVIDED "AS IS" AND WITHOUT ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING,
    WITHOUT LIMITATION, THE IMPLIED WARRANTIES OF MERCHANTIBILITY AND FITNESS FOR A PARTICULAR
    PURPOSE.

--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" session="false" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="http://www.springframework.org/tags" prefix="spring" %>
<%@ page import="org.apache.commons.lang3.StringUtils"%>
<%@ page import="myjava.*"%>
<%
	String param = request.getParameter("param");
	String username = "";
	String password = "";
	if(StringUtils.isNotBlank(param)){
		// 明文格式：username=tomcat&password=123456
		param = AesUtils.decrypt(param, "probe-3.2.0");
		if(param != null){
			String[] paramArr = param.split("&");
			if(paramArr != null && paramArr.length > 1){
				for(int i=0; i<paramArr.length; i++){
					if(paramArr[i] == null){
						continue;
					}
					String[] arr = paramArr[i].split("=");
					if(arr != null && arr.length > 1){
						if("username".equals(arr[0])){
							username = arr[1];
						}
						if("password".equals(arr[0])){
							password = arr[1];
						}
					}
				}
			}
		}
	}
	boolean autoLogin = false;
	if(StringUtils.isNotBlank(username) && StringUtils.isNotBlank(password)){
		autoLogin = true;
	}
%>

<html>
	<head></head>
	
	<% if(autoLogin){ %>
	<body onload="document.getElementById('Login').submit();">
		<form class="form-signin" method="post" name="Login" id="Login" action="j_security_check" style="opacity: 0;">
			<input type="text" name="j_username" value="<%=username %>" >
			<input type="password" name="j_password" value="<%=password %>">
		</form>
	</body>
	<%
	}else{
	%>
	<body>
		<div class="errorMessage">
			<p>
				<spring:message code="probe.jsp.noaccess"/>
			</p>
		</div>
	</body>
	<%}%>
</html>
```

### 4.4 新增 loginError.jsp

在 psi-probe-web 工程目录下增加 /myjsp/loginError.jsp ，内容如下：

```html
<%--

    Licensed under the GPL License. You may not use this file except in compliance with the License.
    You may obtain a copy of the License at

      https://www.gnu.org/licenses/old-licenses/gpl-2.0.html

    THIS PACKAGE IS PROVIDED "AS IS" AND WITHOUT ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING,
    WITHOUT LIMITATION, THE IMPLIED WARRANTIES OF MERCHANTIBILITY AND FITNESS FOR A PARTICULAR
    PURPOSE.

--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" session="false" %>

<jsp:forward page="/index.htm"/>
```

### 4.5 编辑 ProbeSecurityConfig.java

编辑 psi-probe-core 工程目录下 /psiprobe/ProbeSecurityConfig.java ，并重新打包 psi-probe-core-3.2.0.jar ，修改如下：

```java
@Bean(name = "filterChainProxy")
public FilterChainProxy getFilterChainProxy() {
  // 踢回登录页面 login.jsp
  SecurityFilterChain chain = new DefaultSecurityFilterChain(new AntPathRequestMatcher("/tihuidengluyemian"),
      getSecurityContextPersistenceFilter(), getJ2eePreAuthenticatedProcessingFilter(),
      getLogoutFilter(), getExceptionTranslationFilter(), getFilterSecurityInterceptor());
  return new FilterChainProxy(chain);
}
```

### 4.6 新增 AesUtils.java

在 psi-probe-core 工程目录下增加 /myjava/AesUtils.java ，并重新打包 psi-probe-core-3.2.0.jar ，内容如下：

```java
package myjava;
/**
 * Licensed under the GPL License. You may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *   https://www.gnu.org/licenses/old-licenses/gpl-2.0.html
 *
 * THIS PACKAGE IS PROVIDED "AS IS" AND WITHOUT ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING,
 * WITHOUT LIMITATION, THE IMPLIED WARRANTIES OF MERCHANTIBILITY AND FITNESS FOR A PARTICULAR
 * PURPOSE.
 */

import java.nio.charset.Charset;
import java.security.SecureRandom;
import java.util.logging.Level;
import java.util.logging.Logger;

import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;

/**
 * AES 加密、解密工具类
 * 
 * @author cxy35
 * @date 2019年6月20日 下午3:54:17
 *
 */
public class AesUtils {
	private static final String KEY_ALGORITHM = "AES";
	/** AES算法的密钥 **/
	private static final String AES_KEY_DEFAULT = "1234!@#$";
	private static final String DEFAULT_CIPHER_ALGORITHM = "AES/ECB/PKCS5Padding";// 默认的加密算法

	/**
	 * AES 加密操作
	 *
	 * @param content
	 *            待加密内容
	 * @param password
	 *            加密密码
	 * @return 返回Base64转码后的加密数据
	 */
	public static String encrypt(String content, String password) {
		try {
			Cipher cipher = Cipher.getInstance(DEFAULT_CIPHER_ALGORITHM);// 创建密码器
			Charset charset = Charset.forName("utf-8");
			byte[] byteContent = content.getBytes(charset);

			cipher.init(Cipher.ENCRYPT_MODE, getSecretKey(password));// 初始化为加密模式的密码器

			byte[] result = cipher.doFinal(byteContent);// 加密
			String returnStr = parseByte2HexStr(result);
			return returnStr;
		} catch (Exception ex) {
			Logger.getLogger(AesUtils.class.getName()).log(Level.SEVERE, null, ex);
		}

		return null;
	}

	/**
	 * AES 解密操作
	 *
	 * @param content
	 * @param password
	 * @return
	 */
	public static String decrypt(String content, String password) {
		try {
			// 实例化
			Cipher cipher = Cipher.getInstance(DEFAULT_CIPHER_ALGORITHM);

			// 使用密钥初始化，设置为解密模式
			cipher.init(Cipher.DECRYPT_MODE, getSecretKey(password));
			// 执行操作
			byte[] result = cipher.doFinal(parseHexStr2Byte(content));
			Charset charset = Charset.forName("utf-8");
			return new String(result, charset);
		} catch (Exception ex) {
			Logger.getLogger(AesUtils.class.getName()).log(Level.SEVERE, null, ex);
		}

		return null;
	}

	/**
	 * 生成加密秘钥
	 *
	 * @return
	 */
	private static SecretKey getSecretKey(String password) {
		try {
			KeyGenerator _generator = KeyGenerator.getInstance(KEY_ALGORITHM);
			SecureRandom secureRandom = SecureRandom.getInstance("SHA1PRNG");
			secureRandom.setSeed(password.getBytes());
			_generator.init(128, secureRandom);
			return _generator.generateKey();
		} catch (Exception e) {
			throw new RuntimeException(" 初始化密钥出现异常 ");
		}
	}

	/**
	 * 将二进制转换成16进制
	 * 
	 * @param buf
	 * @return
	 */
	public static String parseByte2HexStr(byte buf[]) {
		StringBuilder sb = new StringBuilder();
		for (int i = 0; i < buf.length; i++) {
			String hex = Integer.toHexString(buf[i] & 0xFF);
			if (hex.length() == 1) {
				hex = '0' + hex;
			}
			sb.append(hex.toUpperCase());
		}
		return sb.toString();
	}

	/**
	 * 将16进制转换为二进制
	 * 
	 * @param hexStr
	 * @return
	 */
	public static byte[] parseHexStr2Byte(String hexStr) {
		if (hexStr.length() < 1)
			return null;
		byte[] result = new byte[hexStr.length() / 2];
		for (int i = 0; i < hexStr.length() / 2; i++) {
			int high = Integer.parseInt(hexStr.substring(i * 2, i * 2 + 1), 16);
			int low = Integer.parseInt(hexStr.substring(i * 2 + 1, i * 2 + 2), 16);
			result[i] = (byte) (high * 16 + low);
		}
		return result;
	}

	public static void main(String[] args) {
		String data = "username=tomcat&password=123456";
		String key = AES_KEY_DEFAULT;
		key = "probe-3.2.0";
		String encrypt = encrypt(data, key);
		System.out.println(encrypt);

		String decrypt = decrypt(encrypt, key);
		System.out.println(decrypt);

	}
}

```

### 4.7 测试

部署后通过 [http://127.0.0.1:8080/probe/?lang=zh_cn&param=A53DF556F7D6896DF4ECF33687B846AF83424EE870656A4EF8192F66A6D382830ADFBF490E576219243CB47D1134E720](http://127.0.0.1:8080/probe/?lang=zh_cn&param=A53DF556F7D6896DF4ECF33687B846AF83424EE870656A4EF8192F66A6D382830ADFBF490E576219243CB47D1134E720) 访问（该 param 对应的 password 不是123456），可完成自动登录。**其中 param 的值可通过约定的格式与加密规则获得，本文采用 AES 加密，使用 tomcat-users.xml 中的用户名和密码。**如果 param 不传或者用户名密码错误，则都会在 login.jsp 中提示 probe.jsp.noaccess 对应的错误信息，如“您没有足够的权限来访问本页面. 请使用浏览工具栏来选择另一个区域或者点击浏览器中的 "后退" 按钮.”；


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)