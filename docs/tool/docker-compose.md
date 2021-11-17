## 简介

通过前面几篇文章的学习，我们可以通过 `Dockerfile` 文件让用户很方便的定义一个单独的应用容器。然而，在日常工作中，经常会碰到需要多个容器相互配合来完成某项任务的情况，或者开发一个 `Web` 应用，除了 `Web` 服务容器本身，还需要数据库服务容器、缓存容器，甚至还包括负载均衡容器等等。

`Docker Compose` 恰好满足了这样的需求，它是用于定义和运行多容器 `Docker` 应用程序的工具。通过 `Compose`，您可以使用 `YAML` 文件来配置应用程序所需要的服务。然后使用一个命令，就可以通过 `YAML` 配置文件创建并启动所有服务。

`Docker Compose` 项目是 `Docker` 官方的开源项目，来源于之前的 `Fig` 项目，使用 `Python` 语言编写。负责实现对 `Docker` 容器集群的快速编排。项目地址为：[https://github.com/docker/compose/releases](https://github.com/docker/compose/releases)

`Docker Compose` 使用的三个步骤为：

- 使用 `Dockerfile` 文件定义应用程序的环境；
- 使用 `docker-compose.yml` 文件定义构成应用程序的服务，这样它们可以在隔离环境中一起运行；
- 执行 `docker-compose up` 命令来创建并启动所有服务。

## 安装

官方文档：[https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

```bash
# 下载二进制文件来使用
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# 因为 Docker Compose 存放在 GitHub，可能不太稳定。可以通过 DaoCloud 加速下载
# curl -L https://get.daocloud.io/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# 授权
# 将可执行权限应用于该二进制文件
sudo chmod +x /usr/local/bin/docker-compose

# 测试
docker-compose --version

# 卸载
rm -f /usr/local/bin/docker-compose
```

## docker-compose.yml 文件详解

官方文档：[https://docs.docker.com/compose/compose-file/](https://docs.docker.com/compose/compose-file/)

`Docker Compose` 允许用户通过 `docker-compose.yml` 文件（`YAML` 格式）来定义一组相关联的容器为一个工程（`project`）。一个工程包含多个服务（`service`），每个服务中定义了创建容器（`container`）时所需的镜像、参数、依赖等。

`Docker Compose` 模板文件我们需要关注的顶级配置有 `version、services、networks、volumes` 几个部分，除 `version` 外，其他几个顶级配置下还有很多下级配置，后面也会详细给大家介绍，先来看看这几个顶级配置都什么意思：

`version`：描述 `Compose` 文件的版本信息，当前最新版本为 `3.8`，对应的Docker版本为 `19.03.0+`
`services`：定义服务，可以多个，每个服务中定义了创建容器时所需的镜像、参数、依赖等
`networkds`：定义网络，可以多个，根据 `DNS server` 让相同网络中的容器可以直接通过容器名称进行通信
`volumes`：数据卷，用于实现目录挂载

示例：

```yml
# 描述 Compose 文件的版本信息
version: "3.8"

# 定义服务，可以多个
services:
  mysql: # 服务名称，容器与容器之间可以用服务名称为域名进行访问，比如在 zmall-admin 服务中可以通过 jdbc:mysql//mysql:3306 这个地址来访问 mysql 这个服务。
    image: mysql:5.7 # 指定运行的镜像名称
    container_name: mysql # 配置容器名称，默认为"工程名称_服务条目名称_序号"
    ports: # 指定宿主机和容器的端口映射（HOST:CONTAINER）
      - "3306:3306"
    volumes: # 将宿主机的文件或目录挂载到容器中（HOST:CONTAINER）
      - /home/mysql/conf:/etc/mysql/conf.d
      - /home/mysql/log:/var/log/mysql
      - /home/mysql/data:/var/lib/mysql
    environment: # 配置环境变量
      - MYSQL_ROOT_PASSWORD=123456
    networks: # 非必须，配置容器连接的网络，引用顶级 networks 下的条目
      - mysql-net
  zmall-admin:  # 服务2名称
    image: zmall/zmall-admin:1.0
    container_name: zmall-admin
    ports:
      - 8080:8080
    volumes:
      - /etc/localtime:/etc/localtime
      - /home/zmall/zmall-admin/logs:/var/logs
    environment:
      - 'TZ="Asia/Shanghai"'
    external_links:
      - mysql:db # 可以用 db 这个域名访问 mysql 服务
# 定义网络，可以多个。如果不声明，默认会创建一个网络名称为"工程名称_default"的 bridge 网络
networks:
  mysql-net: # 一个具体网络的条目名称
    name: mysql-net # 网络名称，默认为"工程名称_网络条目名称"
    driver: bridge # 网络模式，默认为 bridge
```

## 常用命令

官方文档：[https://docs.docker.com/compose/reference/overview/](https://docs.docker.com/compose/reference/overview/)

`docker-compose [-f <arg>...] [options] [COMMAND] [ARGS...]`

部分命令选项如下：

`-f，--file`：指定使用的 Compose 模板文件，默认为 docker-compose.yml，可以多次指定，指定多个 yml；
`-p, --project-name`：指定工程名称，默认使用 docker-compose.yml 文件所在目录的名称；
`-v`：打印版本并退出；
`--log-level`：定义日志等级（DEBUG, INFO, WARNING, ERROR, CRITICAL）。

### 查看容器

```bash
# 列出工程中所有服务的容器
docker-compose ps
# 列出工程中指定服务的容器
docker-compose ps mysql
```

### 启动与停止容器

```bash
# 前台启动
docker-compose up
# 后台启动
docker-compose up -d
# -f 指定使用的 Compose 模板文件，默认为 docker-compose.yml，可以多次指定，指定多个 yml
docker-compose -f docker-compose.yml up -d

# 停止工程中所有服务的容器
docker-compose stop
# 停止工程中指定服务的容器
docker-compose stop mysql
```


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)