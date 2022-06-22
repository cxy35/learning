手把手带你在 Linux 上安装 `Nginx` 。
<!-- more -->

## 1 准备工作

### 1.1 下载安装包

- 访问 [http://nginx.org/en/download.html](http://nginx.org/en/download.html) 下载对应**稳定版（Stable version）**的安装包，如：`nginx-1.22.0.tar.gz`。
- 上传到服务器的 `/usr/local/mydata/temp` 目录下。如果没有，则手动新建。

### 1.2 安装相关依赖环境

```bash
# 根据系统实际情况选择安装
# gcc 环境
# pcre 库：一个 Perl 库，包括 perl 兼容的正则表达式库。 Nginx 的 http 模块使用 pcre 来解析正则表达式
# zlib 库：提供了很多种压缩和解压缩的方式。 Nginx 使用 zlib 对 http 包的内容进行 gzip
# openssl 库：一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。 Nginx 不仅支持 http 协议，还支持 https （即在 ssl 协议上传输 http ）

# 下面是全部安装
yum -y install gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel make libtool
```

## 2 安装

### 2.1 解压

```bash
# 其中 nginx-1.22.0.tar.gz 换成实际的名称
cd /usr/local/mydata/temp
tar -xvzf nginx-1.22.0.tar.gz
```

### 2.2 编译安装

```bash
# 进入 nginx-1.22.0 目录，其中 nginx-1.22.0 换成实际的名称
cd /usr/local/mydata/temp/nginx-1.22.0

# 执行 cofigure 命令创建一个 makeFile 文件，如果没有，编译的时候会报错
# \ 表示命令还没有输入完，换行的意思
# --prefix=/usr/local/mydata/soft/nginx 指定 Nginx 安装目录
./configure \
--prefix=/usr/local/mydata/soft/nginx \
--error-log-path=/usr/local/mydata/soft/nginx/logs/error.log \
--http-log-path=/usr/local/mydata/soft/nginx/logs/access.log \
--pid-path=/usr/local/mydata/soft/nginx/logs/nginx.pid \
--lock-path=/usr/local/mydata/soft/nginx/logs/nginx.lock \
--http-client-body-temp-path=/usr/local/mydata/soft/nginx/temp/client-body \
--http-proxy-temp-path=/usr/local/mydata/soft/nginx/temp/proxy \
--http-fastcgi-temp-path=/usr/local/mydata/soft/nginx/temp/fastcgi \
--http-uwsgi-temp-path=/usr/local/mydata/soft/nginx/temp/uwsgi \
--http-scgi-temp-path=/usr/local/mydata/soft/nginx/temp/scgi \
--with-http_stub_status_module \
--with-http_ssl_module \
--with-http_gzip_static_module \
--with-file-aio \
--with-http_realip_module

# 增加第三方模块（需要提前准备模块源码，供编译时用）
# \
# --add-module=/usr/local/mydata/soft/nginx/nginx-upstream-fair-master \
# --add-module=/usr/local/mydata/soft/nginx/nginx_upstream_check_module-master

编译时可能会提示缺少第三方库，更多问题参考 `Nginx 常见问题` 这篇文章。

# 创建上述相关目录
mkdir -p /usr/local/mydata/soft/nginx/{logs,temp}

# 编译安装
make && make install

# 执行完成之后，在 Nginx 安装目录下( /usr/local/mydata/soft/nginx )多了几个目录：conf/html/logs/sbin/temp

# 切换到 Nginx 安装目录
cd /usr/local/mydata/soft/nginx

# 查看 Nginx 版本
/usr/local/mydata/soft/nginx/sbin/nginx -v
# nginx version: nginx/1.22.0

# 查看 Nginx 安装时的配置参数
/usr/local/mydata/soft/nginx/sbin/nginx -V
# configure arguments: --prefix=/usr/local/mydata/soft/nginx --error-log-path=/usr/local/mydata/soft/nginx/logs/error.log --http-log-path=/usr/local/mydata/soft/nginx/logs/access.log --pid-path=/usr/local/mydata/soft/nginx/logs/nginx.pid --lock-path=/usr/local/mydata/soft/nginx/logs/nginx.lock --http-client-body-temp-path=/usr/local/mydata/soft/nginx/temp/client-body --http-proxy-temp-path=/usr/local/mydata/soft/nginx/temp/proxy --http-fastcgi-temp-path=/usr/local/mydata/soft/nginx/temp/fastcgi --http-uwsgi-temp-path=/usr/local/mydata/soft/nginx/temp/uwsgi --http-scgi-temp-path=/usr/local/mydata/soft/nginx/temp/scgi --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module --with-file-aio --with-http_realip_module

# 启动
/usr/local/mydata/soft/nginx/sbin/nginx

# 检查是否启动
ps -ef|grep nginx
# root     26063     1  0 14:14 ?        00:00:00 nginx: master process /usr/local/mydata/soft/nginx/sbin/nginx
# nobody   26064 26063  0 14:14 ?        00:00:00 nginx: worker process      
# root     26066 22818  0 14:15 pts/1    00:00:00 grep nginx

# 删除临时文件
rm -rf /usr/local/mydata/temp/nginx-1.22.0

# 关闭
/usr/local/mydata/soft/nginx/sbin/nginx -s stop
# /usr/local/mydata/soft/nginx/sbin/nginx -s quit

# 重启
/usr/local/mydata/soft/nginx/sbin/nginx -s reopen

# 重新载入配置文件 nginx.conf ，有失效的风险
/usr/local/mydata/soft/nginx/sbin/nginx -s reload

# 检查配置文件 nginx.conf 的正确性
/usr/local/mydata/soft/nginx/sbin/nginx -t
```

启动后，访问 http://127.0.0.1 验证，效果如下：

![](https://oscimg.oschina.net/oscnet/up-2817c8f57921388ed8b69729f77eff74a1f.png)

如果无法访问，排查下防火墙端口是否开放。

## 3 配置

```bash
cp /usr/local/mydata/soft/nginx/conf/nginx.conf /usr/local/mydata/soft/nginx/conf/nginx.conf.bak
echo '' > /usr/local/mydata/soft/nginx/conf/nginx.conf
vi /usr/local/mydata/soft/nginx/conf/nginx.conf
```

新增如下配置：

```bash
user root;
worker_processes  2;

events {
    multi_accept on;
    worker_connections  20480;
}

http {

    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  120;

    proxy_headers_hash_max_size 51200;
    proxy_headers_hash_bucket_size 6400;

    client_header_buffer_size 512k;
    large_client_header_buffers 4 512k;

    gzip  on;

    # upstream szzlApi {
    #     server 127.0.0.1:9070 weight=1;
    # }

    server {
        listen       80;
        server_name  localhost-80;
      
        proxy_connect_timeout 300s;
        proxy_read_timeout 300s;
        proxy_send_timeout 300s;

        if ($time_iso8601 ~ '(\d{4}-\d{2}-\d{2})') {
            set $date_yyyy_MM_dd $1;
        }

        access_log off;
        # access_log logs/access-$date_yyyy_MM_dd.log combined;

        location / {
            root   mydata;
        }

        location ~*/szzlApi/ {
            proxy_pass http://127.0.0.1:9070;
            # proxy_pass http://szzlApi;
            proxy_set_header Host $host:$server_port;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Remote_addr $remote_addr;
        }
        
        # redirect server error pages to the static page /50x.html
        error_page   404              /404.html;
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

执行：`/usr/local/mydata/soft/nginx/sbin/nginx -s reload` 重新加载，使配置生效。


---

- [Nginx 教程合集](https://mp.weixin.qq.com/s/TdLki2vnjW4hKUz_BgzEHg)（微信左下方**阅读全文**可直达）。
- [Nginx 参数配置优化](https://mp.weixin.qq.com/s/wS-ly5O_xSJbVzJ24_yAKQ)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)