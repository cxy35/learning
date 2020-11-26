---
title: Apache 实现 Tomcat 集群 - mod_proxy（新版本）
date: 2018-05-09 11:52:00
categories: Apache
tags: [Apache, Tomcat, 集群, mod_proxy]
toc: true
---
使用 Apache 实现 Tomcat 集群，采用 mod_proxy 组件（新版本）。
<!-- more -->

## 1 准备工作

- Apache 版本：httpd-2.4.23-x64
- 下载地址：[http://httpd.apache.org/download.cgi](http://httpd.apache.org/download.cgi)

---

- Tomcat 版本：apache-tomcat-8.0.53-windows-x64
- 下载地址：[http://tomcat.apache.org/download-70.cgi](http://tomcat.apache.org/download-70.cgi)

- tomcat1（http-8080，ajp-9009，webapp/test）
- tomcat2（http-8081，ajp-9010，webapp/test）
- tomcat3（http-8082，ajp-9011，webapp/test）

安装并分别启动成功。

## 2 配置 Apache

### 2.1 配置 httpd.conf

编辑 Apache 根目录下的 conf/httpd.conf 文件，配置如下：

```bash
# 加载相关模块
##### modify #####
LoadModule access_compat_module modules/mod_access_compat.so

##### modify #####
LoadModule info_module modules/mod_info.so

##### modify #####
LoadModule lbmethod_bybusyness_module modules/mod_lbmethod_bybusyness.so
LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so
LoadModule lbmethod_bytraffic_module modules/mod_lbmethod_bytraffic.so
LoadModule lbmethod_heartbeat_module modules/mod_lbmethod_heartbeat.so

##### modify #####
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_ajp_module modules/mod_proxy_ajp.so
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
LoadModule proxy_connect_module modules/mod_proxy_connect.so
# LoadModule proxy_express_module modules/mod_proxy_express.so
# LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so
LoadModule proxy_ftp_module modules/mod_proxy_ftp.so
# LoadModule proxy_html_module modules/mod_proxy_html.so
LoadModule proxy_http_module modules/mod_proxy_http.so

##### modify #####
LoadModule slotmem_shm_module modules/mod_slotmem_shm.so

##### modify #####
LoadModule status_module modules/mod_status.so
```

```bash
# 增加相关配置
##### modify #####
ProxyRequests Off
# Header add Set-Cookie "ROUTEID=.%{BALANCER_WORKER_ROUTE}e; path=/" env=BALANCER_ROUTE_CHANGED

# ---------- apache 监控-开始 -----------
# 过滤 server-stauts 监控页面，需要放在 <Proxy balancer://cluster> 前面
ProxyPass /server-status ! 
# 过滤 balancer-manager 监控页面
ProxyPass /balancer-manager ! 

# 设置 server-stauts 监控页面
<Location /server-status> 
	SetHandler server-status
	Order Deny,Allow
	Deny from all
	Allow from 127.0.0.1
</Location>

# 设置 balancer-manager 监控页面
<Location /balancer-manager> 
	SetHandler balancer-manager
	Order Deny,Allow
	Deny from all
	Allow from 127.0.0.1
</Location>
# ---------- apache监控-结束 ----------

ProxyPassReverse / balancer://cluster/
<Proxy balancer://cluster>
	BalancerMember ajp://127.0.0.1:9009 loadfactor=1 route=tomcat1 retry=30
	BalancerMember ajp://127.0.0.1:9010 loadfactor=1 route=tomcat2 retry=30
	
	# 负载均衡方式，默认为 byrequests （进行加权请求计数）
	ProxySet lbmethod=byrequests
</Proxy>

# 请求映射过滤规则配置 
# 过滤静态资源， apache 直接返回，不到 tomcat 请求
ProxyPassMatch ^/?[\S]*/test/static !

ProxyPass / balancer://cluster/ stickysession=JSESSIONID|jsessionid scolonpathdelim=On

<IfModule mpm_winnt_module>
    ThreadLimit	4000
    ThreadsPerChild      3840
    MaxRequestsPerChild    0
</IfModule>
AddDefaultCharset off
Timeout 60
KeepAlive On
KeepAliveTimeout 20
MaxKeepAliveRequests 500
```

### 2.2 静态资源

在 Apache 根目录下的 htdocs 文件夹下增加 test/static/common/images、css、js 等静态文件。

```bash
# 静态资源 apache 直接返回，不到 tomcat 请求
ProxyPassMatch ^/\S+/static !
```

## 3 配置 Tomcat

编辑 Tomcat 根目录下的 conf/server.xml 文件，修改配置：

```xml
<!-- 修改 AJP 协议端口，与上面的 worker.properties 中 port 对应 -->
<Connector port="9009" protocol="AJP/1.3" redirectPort="9443"/>

<!-- 添加 jvmRoute ，与上面的 httpd.conf 中对应，集群时候用 -->
<Engine defaultHost="localhost" name="Catalina" jvmRoute="tomcat1">
```

## 4 测试

在 Tomcat 根目录下的 webapps 下新建 test 目录，再在里面新建 test.jsp 文件：

```html
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<%@page import="java.util.*"%>

<html>
<head>
    <title>Tomcat集群测试</title>
	<meta http-equiv="pragma" content="no-cache">
	<meta http-equiv="cache-control" content="no-cache">
</head>
<body>
<%  
out.println(new Date() + "<br/>");
out.println("[tomcat1:8080]" + "<br/>");
String sessionid = request.getSession().getId();
out.println("sessionid: " + sessionid + "<br/>");
System.out.println("sessionid: " + sessionid);
out.println("=============================================<br/>");

Enumeration enu = request.getHeaderNames();  
while(enu.hasMoreElements()){  
	String key = (String)enu.nextElement();  
	out.println(key + ": " + request.getHeader(key) + "<br/>");  
}
out.println("=============================================<br/>");

out.println("request.remote: "+request.getRemoteAddr()+":"+request.getRemotePort() + "<br/>");
out.println("request.local: "+request.getLocalAddr()+":"+request.getLocalPort() + "<br/>");
out.println("=============================================<br/>");
%>
</body>
</html>
```

再在里面新建 WEB-INF/web.xml 文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/j2ee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_5.xsd" version="2.5">
	<display-name>test</display-name>
	<distributable />
</web-app>
```

test 目录如下：

![](https://oscimg.oschina.net/oscnet/6dfda68ccfacece83fbdf55dd516af6e322.jpg)

- 通过 [http://127.0.0.1/test/test.jsp](http://127.0.0.1/test/test.jsp) 访问，查看页面与控制台。
- 通过 [http://127.0.0.1/test/static/common/images/1.png](http://127.0.0.1/test/static/common/images/1.png) 访问（ apache 和 tomcat 都有），查看静态资源与浏览器 Network 请求，最终访问 apache 。
- 通过 [http://127.0.0.1/test/static/common/images/2.png](http://127.0.0.1/test/static/common/images/2.png) 访问（ apache 有， tomcat 没有），查看静态资源与浏览器 Network 请求，最终访问 apache 。

## 5 其他说明

### 5.1 mod_jk 与 mod_proxy

- **Apache2.2 之前，主要用 mod_jk 组件，也比较稳定。**
- **Apache2.2 之后，可用 Apache 自带的 mod_proxy 组件。**

### 5.2 Apache 日志

Apache 根目录下的 logs 文件夹下：

- access.log：用于记录 Apache 接收到请求以及响应状态的日志。
- error.log：用于记录 Apache 的运行错误。
- httpd.pid：用于记录进程 pid 。
- mod_jk.log：用于记录请求转发给 Tomcat 的日志。

### 5.3 Tomcat 日志

Tomcat 根目录下的 logs 文件夹下：

- localhost_access_log.日期.txt：用于记录 Tomcat 接收到的请求以及响应的状态等，作用与 Apache 的 access.log 类似。
- catalina.日期.log：用于记录 Tomcat 启动时候控制台的一些信息以及服务端错误信息。
- localhost.日期.log：用于记录站点访问信息， Tomcat 下内部代码丢出的日志。

### 5.4 其他

Apache 根目录下的 htdocs 文件夹的作用与 Tomcat 的 webapps 类似。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)