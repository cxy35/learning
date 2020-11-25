---
title: Nginx 实现 Tomcat 集群
date: 2018-05-19 15:37:37
categories: Nginx
tags: [Nginx, Tomcat, 集群]
toc: true
---
使用 Nginx 实现 Tomcat 集群。
<!-- more -->

## 1 准备工作

- Nginx 版本：nginx-1.16.0 ，推荐使用 linux 版本。
- 下载地址：[http://nginx.org/en/download.html](http://nginx.org/en/download.html)
- 安装并启动成功。

---

- Tomcat 版本：apache-tomcat-8.0.53-windows-x64
- 下载地址：[http://tomcat.apache.org/download-70.cgi](http://tomcat.apache.org/download-70.cgi)

- tomcat1（http-8080，webapp/test）
- tomcat2（http-8081，webapp/test）
- tomcat3（http-8082，webapp/test。包含静态资源 test/static/...，关于文件的上传和下载的请求会全部转发到这里，可以理解为一个文件服务器）
- 安装并分别启动成功。

## 2 配置 Nginx

### 2.1 配置 nginx.conf

编辑 Nginx 根目录下的 conf/nginx.conf 文件，配置如下（具体的参数配置说明和优化见下文）：

```bash
worker_processes  1;

events {
    multi_accept on;
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  120;

    upstream tomcat_test {
        server 127.0.0.1:8080 weight=1;
        server 127.0.0.1:8081 weight=1;
    }

    upstream tomcat_test_static {
        server 127.0.0.1:8082 weight=1;
    }

    server {
        listen       80;
        server_name  localhost;

	    location ~*/test/static/ {
            proxy_pass http://tomcat_test_static;
        }

        location / {
	        proxy_pass http://tomcat_test;
	        proxy_redirect default;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_next_upstream http_502 http_504 error timeout invalid_header;
        }
    }
}
```

### 2.2 静态资源

```bash
# 静态资源服务器 location，正则匹配，~为区分大小写，~*为不区分大小写
location ~*/test/static/ {
    # 方法1
proxy_pass http://tomcat_test_static;
# 方法2：指定相对路径，相对 nginx 安装目录下的 mydata 为根目录
    # 如 http://127.0.0.1/test/static/common/images/1.png 会映射到 nginx 安装目录 /mydata/test/static/common/images/1.png
    # root mydata;
    # 方法3：指定绝对路径，注意 windows 系统下分隔符用 /
    # 如 http://127.0.0.1/test/static/common/images/1.png 会映射到 D:/apache-tomcat-8.0.53-8082/webapps/test/static/common/images/1.png
    # root D:/apache-tomcat-8.0.53-8082/webapps;

    # 开启目录浏览权限，默认是 off
    # autoindex on;
}
```

### 2.3 参数配置优化

具体配置参考： [Nginx 参数配置优化](https://mp.weixin.qq.com/s/wS-ly5O_xSJbVzJ24_yAKQ)

## 3 配置 Tomcat

编辑 Tomcat 根目录下的 conf/server.xml 文件，修改配置：

```xml
<!-- 添加 jvmRoute ，用于标识该 tomcat -->
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
- 通过 [http://127.0.0.1/test/static/common/images/1.png](http://127.0.0.1/test/static/common/images/1.png) 访问（ tomcat1、tomcat2、tomcat3 都有），查看静态资源与浏览器 Network 请求，最终访问 tomcat3 。
- 通过 [http://127.0.0.1/test/static/common/images/2.png](http://127.0.0.1/test/static/common/images/2.png) 访问（ tomcat3 有， tomcat1 和 tomcat2 没有），查看静态资源与浏览器 Network 请求，最终访问 tomcat3 。

## 5 其他说明

### 5.1 Nginx 日志

Nginx 根目录下的 logs 文件夹下：

- access.log：用于记录 Nginx 接收到请求以及响应状态的日志。
- error.log：用于记录 Nginx 的运行错误。
- nginx.pid：用于记录进程 pid 。

### 5.2 Tomcat 日志

Tomcat 根目录下的 logs 文件夹下：

- localhost_access_log.日期.txt：用于记录 Tomcat 接收到的请求以及响应的状态等，作用与 Apache 的 access.log 类似。
- catalina.日期.log：用于记录 Tomcat 启动时候控制台的一些信息以及服务端错误信息。
- localhost.日期.log：用于记录站点访问信息， Tomcat 下内部代码丢出的日志。

### 5.3 其他

[Nginx 常见问题](https://mp.weixin.qq.com/s/DaESpw7T-NWSfp52bQRx4A)

---

- [Nginx 教程合集](https://mp.weixin.qq.com/s/TdLki2vnjW4hKUz_BgzEHg)（微信左下方**阅读全文**可直达）。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)