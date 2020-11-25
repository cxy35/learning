---
title: Nginx 参数配置优化
date: 2020-03-05 10:46:14
categories: Nginx
tags: [Nginx, 优化]
toc: true
---
Nginx 参数配置优化，提升 Nginx 效率与稳定性，官方文档：[http://nginx.org/en/docs/](http://nginx.org/en/docs/) 。
<!-- more -->

```bash
# 使用的用户和组
# user  nobody;
# user root root;

# 指定工作衍生进程数（一般等于 CPU 的总核数或总核数的两倍，例如两个四核 CPU ，则综合数为 8 。通过命令 ps -ef|grep nginx 可以看出来设置的是几个）
worker_processes  1;
#worker_processes  8;

# 指定错误日志存放的路径，错误日志记录级别可选项为：[debug|info|notice|warn|error|crit]，默认是 crit ，记录的日志数量从 crit 到 debug ，由少到多
# error_log  logs/error.log;
# error_log  logs/error.log  notice;
# error_log  logs/error.log  info;

# 指定 pid 存放的路径
# pid        logs/nginx.pid;

# events settings
events {
    # 设置网路连接序列化，防止惊群现象发生，默认为on 惊群现象：一个网路连接到来，多个睡眠的进程被同时叫醒，但只有一个进程能获得链接，这样会影响系统性能。
    # accept_mutex on;   
    # 设置一个进程是否同时接受多个网络连接，默认为off
    multi_accept on;
    # 使用的网络 I/O 模型， Linux 系统推荐采用 epoll 模型， FreeeBSD 系统推荐采用 kqueue 模型， Windows 系统不指定。
    # use epoll;
    # 允许的连接数
    worker_connections  1024;
}

# 遵循 http 协议的服务器全局设置
http {
    include       mime.types;
    default_type  application/octet-stream;

    # 设置使用的字符集，如果一个网站有多种字符集，请不要随便设置，应让程序员在 HTML 代码中通过 Meta 标签设置
    # charset utf-8;

    # 设置客户端能够上传的文件大小，注意要与应用程序中的文件大小限制兼容。
    # client_max_body_size 30m;

    # 关闭日志记录
    # access_log off;
    # 自定义日志记录格式设置， main 为名字，在 access_log 命令中引用
    # log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
    # 指定日志存放路径，如果想使用默认的 combined 格式记录日志，可以使用 access_log logs/access.log combined; 以下是使用 log_format 自定义的格式记录日志的。
    # access_log  logs/access.log  main;

    sendfile        on;
    # tcp_nopush     on;

    # keepalive_timeout  0;
    keepalive_timeout  120;

    # 开启 gzi p压缩设置（只能在 http 模块中设置）
    # 该指令用于开启或关闭 gzip 模块( on/off )
    # gzip  on;
    # 设置允许压缩的页面最小字节数，页面字节数从 header 头的 content‐length 中进行获取。默认值是 0，不管页面多大都压缩。建议设置成大于 1k 的字节数，小于 1k 可能会越压越大。
    # gzip_min_length 1k;
    # 设置系统获取几个单位的缓存用于存储 gzip 的压缩结果数据流。4 16k 代表以 16k 为单位，安装原始数据大小以 16k 为单位的 4 倍申请内存。
    # gzip_buffers  4 16k;
    # 识别http的协议版本(1. 0/1. 1)
    # gzip_http_version 1.1;
    # gzip 压缩比，1压缩比最小处理速度最快，9 压缩比最大但处理速度最慢(传输快但比较消耗 cpu )
    # gzip_comp_level 2;
    # 匹配 mime 类型进行压缩，无论是否指定,  “text/html” 类型总是会被压缩的。
    # gzip_types application/x-javascript text/css application/xml;
    # 和 http 头有关系，加个 vary 头，给代理服务器用的，有的浏览器支持压缩，有的不支持，所以避免浪费不支持的也压缩，所以根据客户端的 HTTP 头来判断，是否需要压缩。 跟 Squid 等缓存服务有关， on 的话会在 Header 里增加"Vary: Accept‐Encoding"
    # gzip_vary on;
    # IE6 对 Gzip 不怎么友好， 不给它 Gzip 了
    # gzip_disable "MSIE [ 1‐6] \. ";

    # 用于设置如果出现指定的 HTTP 错误状态码，则返回指定的 url 页面
    # error_page  404              /404.html;
    # error_page  500 502 503 504  /50x.html;

    # upstream 负载均衡器 1 设置，用来处理普通请求。
    upstream tomcat_test {
        server 127.0.0.1:8080 weight=1;
        server 127.0.0.1:8081 weight=1;
	
        # 轮询（默认）：每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除。
        # server 127.0.0.1:8080;
        # server 127.0.0.1:8081;

	    # 权重（weight）：指定轮询几率， weight 和访问比率成正比，用于后端服务器性能不均的情况。权重越高，在被访问的概率越大，如下分别是 30%，70%。
        # server 127.0.0.1:8080 weight=3;
        # server 127.0.0.1:8081 weight=7;

        # IP 分配（ip_hash）：每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个后端服务器，可以解决 session 的问题。
	    # ip_hash;
	    # server 127.0.0.1:8080;
        # server 127.0.0.1:8081;

	    # URL 分配（url_hash）：（需要事先安装插件）每个请求按访问地址 url 的 hash 结果分配，能实现同一个 url 访问同一个服务器，也就是根据 url 进行负载，后端服务器为缓存时比较有效。
	    # hash $request_uri;
	    # hash_method crc32;
	    # server 127.0.0.1:8080;
        # server 127.0.0.1:8081;

	    # 响应时间（fair）：（需要事先安装插件）按后端服务器的响应时间来分配请求，响应时间短的优先分配。
	    # fair;
	    # server 127.0.0.1:8080;
        # server 127.0.0.1:8081;

        # server 指令的参数为:
        # 1. down：表示当前的服务暂不参于负载。如：server 127.0.0.1:9080 down;
        # 2. backup：备用服务器，当其它所有非 backup 机器 down 或者忙时，才会请求 backup 机器。如：server 127.0.0.1:9081 backup;
        # 3. weight：设置服务器的权重，默认值是1，权重值越大那么该服务器被访问到的几率就越大。
        # 4. max_fails：允许请求失败的次数，默认为 1（这意味着一发生错误就认为服务器挂掉），如果把 max_fails 设为 0 则表示把这个检查取消。当超过最大次数时，返回 proxy_next_upstream 模块定义的错误。
        # 5. fail_timeout：max_fails 次失败后，暂停的时间，默认是 10 秒。
        # 4 和 5 一般关联使用，举个例子：server 127.0.0.1:8080 max_fails=3 fail_timeout=30s; 表示如果服务器 127.0.0.1:8080 在 30 秒内出现了 3 次错误，那么就认为这个服务器工作不正常，从而在接下来的 30 秒内 nginx 不再去访问这个服务器。
    }

    # upstream 负载均衡器2设置，主要用来处理文件的上传和下载，可以理解为一个文件服务器，所有文件相关的上传和下载都通过这组服务器。
    upstream tomcat_test_static {
        server 127.0.0.1:8082 weight=1;
    }

    # server 虚拟主机设置，可以设置多个：基于 IP 的虚拟主机，基于域名的虚拟主机
    # 虚拟主机-基于域名，反向代理 tomcat_test 和 tomcat_test_static 这两组服务器
    server {
        # 监听的端口
        listen       80;
        # 主机名称
        server_name  localhost;

        # 设置 Nginx 的默认首页文件
        # index index.html index.htm index.jsp index.do;

        # root D:/nginx-1.16.0;
        
        # 配置该虚拟机的字符设置，如果不配置继承自 http 中的 charset 设置
        # charset koi8-r;

        # 访问日志文件设置，如果 server 虚拟机中不设置，则继承 http 模块中的 access_log 的设置
        # if ($time_iso8601 ~ '(\d{4}-\d{2}-\d{2})') {
        #     set $date_yyyy_MM_dd $1;
        # }
        # access_log logs/access-$date_yyyy_MM_dd.log combined;
        # access_log logs/access.log  main;

        # location settings，可以用在 server 节点中。
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

        # 普通服务器 location
        location / {
            # 请求转发的地址，/ 表示拦截到所有的请求
	        proxy_pass http://tomcat_test;
	        # 设置当发生重定向请求时，nginx 自动修正响应头数据（默认是 Tomcat 返回重定向，此时重定向的地址是 Tomcat 的地址，我们需要将之修改    使之成为 Nginx 的地址）
	        proxy_redirect default;
	        # 变量 $host 等于客户端请求头中的 Host 值。
            proxy_set_header Host $host;
            # 后端的 web 服务器可以通过 X-Forwarded-For 获取真实的IP地址， $remote_addr 客户端的 ip 地址
            proxy_set_header X-Forwarded-For $remote_addr;
            
            # HTTP 代理模块 proxy，主要是用来转发请求到其他服务器
            # 如果后端服务器返回 502，504，执行超时等错误，自动将请求转发到 upstream 负载均衡池中的另一台服务器，实现 failover 。
            proxy_next_upstream http_502 http_504 error timeout invalid_header;
        }

        # image expires settings
        # expires 属于 http Header 模块，主要用来 Nginx 返回给用户网页添加附件的 header 信息，可以在 http,server,location 中使用
        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
        {
            expires 30d;
        }

        # js/css/html expires settings
        # expires 属于 http Header 模块，主要用来 Nginx 返回给用户网页添加附件的 header 信息，可以在 http,server,location 中使用
        location ~ .*\.(js|css|html)?$
        {
            expires 2h;
        }

        # 如果 http 模块设置了，则继承。此处设置了则覆盖。
        # error_page  404              /404.html;
        # error_page  500 502 503 504  /50x.html;
        #location = /50x.html {
        #    root   html;
        #}
    }
}
```

---

- [Nginx 教程合集](https://mp.weixin.qq.com/s/TdLki2vnjW4hKUz_BgzEHg)（微信左下方**阅读全文**可直达）。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)