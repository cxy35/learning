---
title: Tomcat 访问日志分析工具 - AWStats
date: 2018-11-23 10:47:11
categories: Tomcat
tags: [Tomcat, 工具, AWStats]
toc: true
---
使用服务器日志分析工具 AWStats 分析 Tomcat 的访问日志。
<!-- more -->

## 1 准备工作

Advanced Web Statistics(AWStats) 是一个免费的功能强大的服务器日志分析工具，它可以告诉你所有的 Web 统计数据，包括访问量、访问者数量、页面、 点击、高峰时段、操作系统、浏览器版本、搜索引擎、关键字、机械访问、无效连接等等。可以工作在大多数服务器上( IIS 5.0+,Apache,Tomcat )，可以从命令行或者 CGI 运行。

---

- Linux 或 windows
- Tomcat：7.0.70
- Java：jdk1.7.0_80(64 bit)
- ActivePerl：5.24.3( linux 系统自带， Win 环境需要额外安装)
- AWStats：7.7

---

- 下载 ActivePerl：[http://www.activestate.com/activeperl/downloads/](http://www.activestate.com/activeperl/downloads/)
- 下载 AWStats：[http://sourceforge.net/projects/awstats/](http://sourceforge.net/projects/awstats/)

## 2 安装及配置 AWStat

将 AWStats 解压或安装，目录如下：

![](https://oscimg.oschina.net/oscnet/3bd87ca63d3ac7988739008c8255458619a.jpg)

在 $TOMCAT_HOME$/webapps 下创建 awstats 文件夹，将 AWStats 解压后目录中的文件拷贝过来，目录如下：

![](https://oscimg.oschina.net/oscnet/0da86cd2629490d34d45e884e44b09d5c84.jpg)

并在 WEB-INF 目录下创建 web.xml 文件，内容如下：

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>    
<web-app xmlns="http://java.sun.com/xml/ns/j2ee"    
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"    
    xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd"    
    version="2.4">    
    
    <servlet>    
        <servlet-name>cgi</servlet-name>    
        <servlet-class>org.apache.catalina.servlets.CGIServlet</servlet-class>    
        <init-param>    
          <param-name>debug</param-name>    
          <param-value>0</param-value>    
        </init-param>    
        <init-param>    
          <param-name>cgiPathPrefix</param-name>    
          <param-value>WEB-INF/cgi-bin</param-value>    
        </init-param>    
         <load-on-startup>5</load-on-startup>    
    </servlet>    
         
    <servlet-mapping>    
        <servlet-name>cgi</servlet-name>    
        <url-pattern>/cgi-bin/*</url-pattern>    
    </servlet-mapping>    
         
    <welcome-file-list>    
        <welcome-file>index.html</welcome-file>    
        <welcome-file>index.htm</welcome-file>    
        <welcome-file>index.jsp</welcome-file>    
   </welcome-file-list>    
    
</web-app>
```

在 cgi-bin 目录下，重命名 awstats.model.conf 为 common.conf ，并创建 awstats.localhost.conf 输入：

```bash
# awstats.xxx.conf：xxxx 为需要监控的站点域名，如果需要监控多个站点，可配置多个文件即可；

Include "common.conf"   

# 配置 tomcat 下的访问日志目录，注意前后缀需要与 tomcat 中的配置一致
LogFile="D:/apache-tomcat-7.0.70-utf8(9081)/logs/localhost_access_log.%yyyy-%mm-%dd.txt"   
# 配置日志格式，需要与 tomcat 访问日志的格式配置匹配
# $TOMCAT_HOME$/conf/server.xml中pattern：配置日志的格式，可设置为 common （默认）或 combined ，也可自行配置格式。
# common：%h %l %u %t &quot;%r&quot; %s %b
# combined：%h %l %u %t %r %s %b %{Referer}i %{User-Agent}i
# pattern="%h %l %u %t &quot;%r&quot; %s %b"
LogFormat = "%host - %logname %time1 %methodurl %code %bytesd"
# pattern="%h %u %t &quot;%r&quot; %s %b %{Referer}i &quot;%{User-Agent}i&quot;;"
# LogFormat =“%host %logname %time1 %methodurl %code %bytesd %referer %uaquot”
# 配置的域名
SiteDomain="localhost"   
# 配置的访问地址
HostAliases="localhost 127.0.0.1"   
# 默认进入 AWStats 的文件
DefaultFile="index.jsp"   
 
# 在 cgi-bin 建立 data 文件夹    
# 此站点监控的统计数据的目录（！！需要手工创建！！）
DirData="data"   
DirCgi="/cgi-bin"   
DirIcons="/awstats/icon"   
 
# 允许在 web 页更新日志，默认为 0（命令行更新）    
# 1 为可在监控页面中点击更新，0 为不允许；
AllowToUpdateStatsFromBrowser=1

```

LogFormat 配置说明如下：

```bash
#   %host             Client hostname or IP address (or Sender host for mail log)  
#   %host_r           Receiver hostname or IP address (for mail log)  
#   %lognamequot      Authenticated login/user with format: "john"  
#   %logname          Authenticated login/user with format: john  
#   %time1            Date and time with format: \[dd/mon/yyyy:hh:mm:ss +0000\] or \[dd/mon/yyyy:hh:mm:ss\]  
#   %time2            Date and time with format: yyyy-mm-dd hh:mm:ss  
#   %time3            Date and time with format: Mon dd hh:mm:ss or Mon dd hh:mm:ss yyyy  
#   %time4            Date and time with unix timestamp format: dddddddddd  
#   %methodurl        Method and URL with format: "GET /index.html HTTP/x.x"  
#   %methodurlnoprot  Method and URL with format: "GET /index.html"  
#   %method           Method with format: GET  
#   %url              URL only with format: /index.html  
#   %query            Query string (used by URLWithQuery option)  
#   %code             Return code status (with format for web log: 999)  
#   %bytesd           Size of document in bytes  
#   %refererquot      Referer page with format: "http://from.com/from.htm"  
#   %referer          Referer page with format: http://from.com/from.htm  
#   %uabracket        User agent with format: \[Mozilla/4.0 (compatible, ...)\]  
#   %uaquot           User agent with format: "Mozilla/4.0 (compatible, ...)"  
#   %ua               User agent with format: Mozilla/4.0_(compatible...)  
#   %gzipin           mod_gzip compression input bytes: In:XXX  
#   %gzipout          mod_gzip compression output bytes & ratio: Out:YYY:ZZpct.  
#   %gzipratio        mod_gzip compression ratio: ZZpct.  
#   %deflateratio     mod_deflate compression ratio with format: (ZZ)  
#   %email            EMail sender (for mail log)  
#   %email_r          EMail receiver (for mail log)  
#   %virtualname      Web sever virtual hostname. Use this tag when same log  
#                     contains data of several virtual web servers. AWStats  
#                     will discard records not in SiteDomain nor HostAliases  
#   %cluster          If log file is provided from several computers (merged by  
#                     logresolvemerge.pl), use this to define cluster id field.  
#   %extraX           Another field that you plan to use for building a  
#                     personalized report with ExtraSection feature (See later).  
#   If your log format has some fields not included in this list, use:  
#   %other            Means another not used field  
#   %otherquot        Means another not used double quoted field
```

## 3 配置Tomcat

修改 $TOMCAT_HOME$/conf/context.xml ，在 Context 节点中追加：privileged="true" 。

修改 $TOMCAT_HOME$/conf/server.xml ，找到 Host ，并修改或追加 VALUE 节点如下：

```xml
<Host appBase="webapps" autoDeploy="true" name="localhost" unpackWARs="true">

        <!-- SingleSignOn valve, share authentication between web applications
             Documentation at: /docs/config/valve.html -->
        <!--
        <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
        -->

        <!-- Access log processes all example.
             Documentation at: /docs/config/valve.html
             Note: The pattern used is equivalent to using pattern="common" -->
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs" pattern="%h %l %u %t &quot;%r&quot; %s %b" prefix="localhost_access_log." suffix=".txt"/>

</Host>
```

patter 配置说明如下：

```bash
    A formatting layout identifying the various information fields from the request and response to be logged, or the word `common` or `combined` to select a standard format. See below for more information on configuring this attribute.

%a - Remote IP address  
%A - Local IP address  
%b - Bytes sent, excluding HTTP headers, or '-' if zero  
%B - Bytes sent, excluding HTTP headers  
%h - Remote host name (or IP address if enableLookups for the connector is false)  
%H - Request protocol  
%l - Remote logical username from identd (always returns '-')  
%m - Request method (GET, POST, etc.)  
%p - Local port on which this request was received. See also %{xxx}p below.  
%q - Query string (prepended with a '?' if it exists)  
%r - First line of the request (method and request URI)  
%s - HTTP status code of the response  
%S - User session ID  
%t - Date and time, in Common Log Format  
%u - Remote user that was authenticated (if any), else '-'  
%U - Requested URL path  
%v - Local server name  
%D - Time taken to process the request, in millis  
%T - Time taken to process the request, in seconds  
%F - Time taken to commit the response, in millis  
%I - Current request thread name (can compare later with stacktraces)
```

## 4 启动及验证

启动 tomcat 后，输入：[http://localhost:8080/awstats/cgi-bin/awstats.pl?config=localhost](http://localhost:8080/awstats/cgi-bin/awstats.pl?config=localhost)  
config 为需要查看的统计站点，与你配置相同即可；效果图如下：

![](https://oscimg.oschina.net/oscnet/2637b0892e709d8c98721d94dacd041b12f.jpg)

手动点击“立即更新“或自己实现定时更新。如可以设置一个 crotab -e 进行配置：0 */10 * * * curl http://localhost:8080/awstats/cgi-bin/awstats.pl?&config=localhost&month=11&year=2018&framename=mainright&update=1


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)