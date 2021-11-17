## 安装

安装详细说明参考官方文档：[https://docs.docker.com/get-docker](https://docs.docker.com/get-docker)，以 CentOS 为例。

```bash
# 安装 yum-utils 包
yum install -y yum-utils

# 设置 Docker 仓库
# 官方地址（比较慢）
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 阿里云地址（国内地址，相对更快）
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 安装 Docker 引擎
yum install docker-ce docker-ce-cli containerd.io

# 查看 Docker 版本信息
docker version

# 启动 docker 服务
systemctl start docker
# 查看 docker 服务状态
systemctl status docker
# 停止 docker 服务
systemctl stop docker
# 重启 docker 服务
systemctl restart docker

# 查看 Docker 的磁盘使用情况
docker system df

# 查看 Docker 的 CPU、内存、网络、IO 等使用情况
# 查看所有容器
docker stats -a
# 查看指定容器：docker stats 容器名称或容器ID
docker stats nginx
```

## 配置镜像

`Docker` 默认拉取镜像是从 [https://hub.docker.com](https://hub.docker.com) 拉取，国外地址拉取的速度比较慢。我们也可以配置国内镜像源，比如阿里云镜像加速器。访问地址 [https://help.aliyun.com/document_detail/60750.html](https://help.aliyun.com/document_detail/60750.html)，进入**容器镜像服务控制台**创建加速器，复制加速器地址，如：https://xxxxxx.mirror.aliyuncs.com。

```bash
# 配置加速器
mkdir -p /etc/docker
vim /etc/docker/daemon.json
{
  "registry-mirrors": ["https://xxxxxx.mirror.aliyuncs.com"]
}

# 重启 Docker Daemon
systemctl daemon-reload
# 重启 docker 服务
systemctl restart docker
```

## 镜像命令

官方文档：[https://docs.docker.com/reference](https://docs.docker.com/reference)

### 查看镜像

```bash
docker images

# REPOSITORY	镜像在仓库中的名称
# TAG	镜像标签
# IMAGE ID	镜像 ID
# CREATED	镜像的创建日期
# SIZE	镜像大小
REPOSITORY           TAG       IMAGE ID       CREATED        SIZE
mysql                5.7       938b57d64674   3 weeks ago    448MB
redis                6.2.6     7faaec683238   5 weeks ago    113MB
nginx                1.20.1    c8d03f6b8b91   4 weeks ago    133MB
nacos/nacos-server   2.0.3     433eb51fef8d   3 months ago   1.05GB
```

这些镜像默认都是存储在 Docker 宿主机的 `/var/lib/docker` 目录下。

### 搜索镜像

```bash
# 从网络中查找需要的镜像：docker search 镜像名称
docker search mysql

# NAME	镜像名称
# DESCRIPTION	镜像描述
# STARS	用户评价
# OFFICIAL	是否为官方构建
# AUTOMATED	Docker Hub 自动构建
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
nginx                             Official build of Nginx.                        15800     [OK]       
jwilder/nginx-proxy               Automated Nginx reverse proxy for docker con…   2093                 [OK]
richarvey/nginx-php-fpm           Container running Nginx + PHP-FPM capable of…   818                  [OK]
jc21/nginx-proxy-manager          Docker container for managing Nginx proxy ho…   273                  
linuxserver/nginx                 An Nginx container, brought to you by LinuxS…   159                  
tiangolo/nginx-rtmp               Docker image with Nginx using the nginx-rtmp…   144                  [OK]
```

### 拉取镜像

拉取镜像就是从中央仓库下载镜像到本地。

```bash
# docker pull 镜像名称:镜像标签
# 如果不声明 tag 镜像标签信息，则默认拉取当前 latest 版本
docker pull nginx:1.20.1
```

### 删除镜像

```bash
# 按镜像ID删除单个镜像：docker rmi 镜像ID
docker rmi c8d03f6b8b91

# 按镜像ID删除多个镜像：docker rmi 镜像ID 镜像ID 镜像ID
# docker rmi c8d03f6b8b91 938b57d64674 7faaec683238

# 按镜像ID删除所有镜像，其中 docker images -q 可以查询到所有镜像的ID
docker rmi docker images -q
```

> 注意：如果通过某个镜像创建了容器，则该镜像无法删除。解决办法：先删除镜像中的容器，再删除该镜像。

## 容器命令

### 查看容器

```bash
# 查看正在运行的容器
docker ps

# CONTAINER ID	容器 ID
# IMAGE	所属镜像
# COMMAND	启动容器时运行的命令
# CREATED	创建时间
# STATUS	容器状态
# PORTS	端口
# NAMES	容器名称
CONTAINER ID   IMAGE                      COMMAND                  CREATED      STATUS             PORTS                                                  NAMES
abe0cf621fed   redis:6.2.6                "docker-entrypoint.s…"   21 seconds ago   Up 20 seconds   0.0.0.0:6379->6379/tcp, :::6379->6379/tcp
a27fdaf5fbd8   nginx:1.20.1               "/docker-entrypoint.…"   2 days ago   Up 2 days          0.0.0.0:80->80/tcp, :::80->80/tcp                      nginx
7fbcd08dd3a7   mysql:5.7                  "docker-entrypoint.s…"   2 days ago   Up 2 days          0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp   mysql
80912a86d5f1   nacos/nacos-server:2.0.3   "bin/docker-startup.…"   2 days ago   Up 2 days          0.0.0.0:8848->8848/tcp, :::8848->8848/tcp              nacos

# 查看所有容器
docker ps -a

# 查看停止的容器
docker ps -f status=exited

# 查看容器的元信息：docker inspect 容器名称或容器ID
docker inspect nginx
```

### 创建启动进入退出容器

`docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`

- `-i`：表示运行容器；
- `-t`：表示容器启动后会进入其命令行。加入这两个参数后，容器创建就能登录进去。即分配一个伪终端；
- `--name`：为创建的容器命名；
- `-v`：表示目录映射关系（前者是宿主机目录，后者是映射到宿主机上的目录），可以使用多个 -v 做多个目录或文件映射。注意：最好做目录映射，在宿主机上做修改，然后共享到容器上；
- `-d`：在 run 后面加上 -d 参数，则会创建一个守护式容器在后台运行（这样创建容器后不会自动登录容器，如果只加 -i -t 两个参数，创建容器后就会自动进容器里）；
- `-p`：表示端口映射，前者是宿主机端口，后者是容器内的映射端口。可以使用多个 -p 做多个端口映射。
- `-P`：随机使用宿主机的可用端口与容器内暴露的端口映射。

```bash
# 创建容器：docker run -di --name 容器名称 -p 宿主机端口:容器内的映射端口 镜像名称:标签
docker run -di --name nginx -p 80:80 nginx:1.20.1

# 进入容器：docker exec -it 容器名称或容器ID /bin/bash
docker exec -it nginx /bin/bash

# 创建并进入容器：docker run -dit --name 容器名称 -p 宿主机端口:容器内的映射端口 镜像名称:标签 /bin/bash
docker run -dit --name nginx -p 80:80 nginx:1.20.1 /bin/bash

# 退出当前容器
exit
```

### 停止与启动容器

```bash
# 停止容器：docker stop 容器名称或容器ID
docker stop nginx

# 启动容器：docker start 容器名称或容器ID
docker start nginx

# 强制停止容器：docker kill 容器名称或容器ID
docker kill nginx
```

### 删除容器

```bash
# 删除指定的容器：docker rm 容器名称或容器ID
docker rm nginx
# 删除多个指定的容器：docker rm 容器名称或容器ID 容器名称或容器ID
docker rm nginx mysql
```

### 文件拷贝

```bash
# 宿主机拷贝到容器：docker cp 需要拷贝的文件或目录 容器名称:容器目录
docker cp /home/nginx/conf nginx:/etc/nginx

# 容器拷贝到宿主机：docker cp 容器名称:容器目录 需要拷贝的文件或目录
docker cp nginx:/etc/nginx /home/nginx/conf
```

### 目录挂载

我们可以在创建容器的时候，将宿主机的目录与容器内的目录进行映射，这样我们就可以通过修改宿主机某个目录的文件从而去影响容器，而且这个操作是双向绑定的，也就是说容器内的操作也会影响到宿主机，实现备份功能。

但是容器被删除的时候，宿主机的内容并不会被删除。如果多个容器挂载同一个目录，其中一个容器被删除，其他容器的内容也不会受到影响。

```bash
# 创建容器时添加 -v 参数，格式为：宿主机目录:容器目录
docker run -di --name nginx -p 80:80 -v /home/nginx/conf:/etc/nginx nginx:1.20.1

# 多目录挂载：docker run -di -v 宿主机目录:容器目录 -v 宿主机目录2:容器目录2 镜像名称:标签
```

> 提示：目录挂载操作可能会出现权限不足的提示。这是因为 CentOS7 中的安全模块 SELinux 把权限禁掉了，在 `docker run` 时通过 `--privileged=true` 给该容器加权限来解决挂载的目录没有权限的问题。

### 查看容器的 IP 地址

```bash
# 查看容器的元信息：docker inspect 容器名称或容器ID
docker inspect nginx

# 直接输出 IP 地址：docker inspect --format='{{.NetworkSettings.IPAddress}}' 容器名称或容器ID
docker inspect --format='{{.NetworkSettings.IPAddress}}' nginx
```

### 查看容器的日志

```bash
# 查看当前全部日志：docker logs 容器名称或容器ID
docker logs nginx

# 动态查看日志：docker logs 容器名称或容器ID -f
docker logs nginx -f
```

## 应用部署

### Nacos

```bash
docker pull nacos/nacos-server:2.0.3
# MODE=standalone表示单机启动
docker run -di --name nacos -p 8848:8848 --env MODE=standalone nacos/nacos-server:2.0.3
```

### Nginx

```bash
docker pull nginx:1.20.1
# 先创建并运行一次容器（为了拷贝配置文件）
docker run -di --name nginx -p 80:80 nginx:1.20.1

mkdir -p /home/nginx
# 将容器内的配置文件拷贝到指定目录
docker cp nginx:/etc/nginx /home/nginx/conf

# 关闭并删除容器
docker stop nginx
docker rm nginx

# 重新创建并运行容器
# -v /home/nginx/html:/usr/share/nginx/html -v /home/nginx/logs:/var/log/nginx
docker run -di --name nginx -p 80:80 -v /home/nginx/conf:/etc/nginx nginx:1.20.1
```

### MySQL

```bash
# 这里默认拉取当前 5.7 的最新版本
docker pull mysql:5.7
mkdir -p /home/mysql/conf /home/mysql/log /home/mysql/data
# -v /home/mysql/conf:/etc/mysql
# -e 设置参数
docker run -di --name mysql -p 3306:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/log:/var/log/mysql -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7

# 进入 MySQL 容器
docker exec -it mysql /bin/bash
```

### Redis

```bash
docker pull redis:6.2.6
# -v /home/redis/data:/data
docker run -di --name redis -p 6379:6379 redis:6.2.6

# 进入 Redis 容器，并使用 redis-cli 命令进行连接
docker exec -it redis redis-cli
```

### RabbitMQ

```bash
docker pull rabbitmq:3.8.25
docker run -di --name rabbitmq -p 15672:15672 -p 5672:5672 rabbitmq:3.8.25

# 进入 RabbitMQ 容器，并开启管理功能
docker exec -it rabbitmq /bin/bash
rabbitmq-plugins enable rabbitmq_management
```

### Elasticsearch

```bash
docker pull elasticsearch:7.14.2
mkdir -p /home/elasticsearch/plugins /home/elasticsearch/data
# 修改虚拟内存区域大小，否则会因为过小而无法启动
sysctl -w vm.max_map_count=262144
# -e 设置参数
docker run -di --name elasticsearch -p 9200:9200 -p 9300:9300 -v /home/elasticsearch/plugins:/usr/share/elasticsearch/plugins -v /home/elasticsearch/data:/usr/share/elasticsearch/data -e "discovery.type=single-node" -e "cluster.name=elasticsearch" elasticsearch:7.14.2

# 进入 Elasticsearch 容器，并安装中文分词器 IKAnalyzer
docker exec -it elasticsearch /bin/bash
elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.14.2/elasticsearch-analysis-ik-7.14.2.zip

# 重启 Elasticsearch 容器
docker restart elasticsearch
```

### Logstash

```bash
docker pull logstash:7.14.2
mkdir -p /home/logstash
# 拷贝配置文件 logstash.conf 到 /home/logstash 目录下，并修改 output 节点下的 Elasticsearch 连接地址为 es:9200
# output {
#   elasticsearch {
#     hosts => "es:9200"
#     index => "mall-%{type}-%{+YYYY.MM.dd}"
#   }
# }

# -e 设置参数
docker run -di --name logstash -p 4560:4560 -p 4561:4561 -p 4562:4562 -p 4563:4563 -v /home/logstash/logstash.conf:/usr/share/logstash/pipeline/logstash.conf --link elasticsearch:es logstash:7.14.2

# 进入 Logstash 容器，并安装 json_lines 插件
docker exec -it logstash /bin/bash
logstash-plugin install logstash-codec-json_lines

# 重启 Logstash 容器
docker restart logstash
```

### Kibana

```bash
docker pull kibana:7.14.2
# -e 设置参数
docker run -di --name kibana -p 5601:5601 --link elasticsearch:es -e "elasticsearch.hosts=http://es:9200" kibana:7.14.2
```

### MongoDB

```bash
docker pull mongo:5.0.3
mkdir -p /home/mongo
docker run -di --name mongo -p 27017:27017 -v /home/mongo/db:/data/db mongo:5.0.3
```

## 构建镜像

```bash
# 使用当前目录的 Dockerfile 创建镜像
docker build -t helloword:1.0.0 .

# 通过 -f Dockerfile 文件的位置创建镜像
docker build -f /home/docker/Dockerfile -t helloword:1.0.0 .
```

---

编写 Dockerfile 文件 `vi Dockerfile`：

```bash
# 该镜像需要依赖的基础镜像
FROM java:8
# 将当前目录下的 jar 包复制到 docker 容器的 /home/zmall 目录下
ADD zmall-admin-1.0-SNAPSHOT.jar /home/zmall
# 运行过程中创建一个 zmall-admin.jar 文件
RUN bash -c 'touch /home/zmall/zmall-admin.jar'
# 声明服务运行在 8080 端口
EXPOSE 8080
# 指定 docker 容器启动时运行 jar 包
ENTRYPOINT ["java", "-jar","/home/zmall/zmall-admin.jar"]
# 指定维护者的名字
MAINTAINER cxy35
```

```bash
cd /home/zmall

# 构建镜像
docker build -f /home/zmall/Dockerfile -t zmall/zmall-admin:1.0 .

# 创建并启动镜像
docker run -di --name zmall-admin -p 8080:8080 zmall/zmall-admin:1.0

# 进入容器
docker exec -it zmall-admin /bin/bash

# 查看镜像构建历史
docker history zmall/zmall-admin:1.0
```


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)