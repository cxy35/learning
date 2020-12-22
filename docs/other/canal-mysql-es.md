通过本文学习如何通过 canal 将 MySQL 数据实时同步到 Elasticsearch。
<!-- more -->

## 1 简介

![](https://oscimg.oschina.net/oscnet/up-35f1c5c1185de32544ac968626f151379d6.png)

`canal [kə'næl]`，译意为水道/管道/沟渠，主要用途是基于 MySQL 数据库增量日志解析，提供增量数据订阅和消费。

基于日志增量订阅和消费的业务包括：

- 数据库镜像
- 数据库实时备份
- 索引构建和实时维护（拆分异构索引、倒排索引等）
- 业务 cache 刷新
- 带业务逻辑的增量数据处理

当前的 canal 支持源端 MySQL 版本包括 5.1.x , 5.5.x , 5.6.x , 5.7.x , 8.0.x

## 2 工作原理

### 2.1 MySQL 主备复制原理

![](https://oscimg.oschina.net/oscnet/up-23f16b581233d27dae3c7b54499deffc4ad.JPEG)

- MySQL master 将数据变更写入二进制日志（binary log, 其中记录叫做二进制日志事件 binary log events，可以通过 show binlog events 进行查看）。
- MySQL slave 将 master 的 binary log events 拷贝到它的中继日志（relay log）。
- MySQL slave 重放 relay log 中事件，将数据变更反映它自己的数据。

### 2.2 canal 工作原理

- canal 模拟 MySQL slave 的交互协议，伪装自己为 MySQL slave ，向 MySQL master 发送 dump 协议。
- MySQL master 收到 dump 请求，开始推送 binary log 给 slave（即 canal）。
- canal 解析 binary log 对象（原始为 byte 流）。

## 3 基本使用

### 3.1 版本约定

本文使用的各组件版本约定如下：

|组件|版本|端口|
|-:|-:|-:|
|MySQL|5.7.29|3306|
|Elasticsearch|7.8.0|9200|
|Kibanba|7.8.0|5601|
|canal.deployer|1.1.15|11111|
|canal.adapter|1.1.15|8081|
|canal.admin|1.1.15|8089|

### 3.2 组件下载与介绍

下载地址：[https://github.com/alibaba/canal/releases](https://github.com/alibaba/canal/releases)

- canal.deployer（canal.server）：可以直接监听 MySQL 的 binlog，把自己伪装成 MySQL 的从库，只负责接收数据，并不做处理。
- canal.adapter：相当于 canal 的客户端，会从 canal.deployer 中获取数据，然后对数据进行同步，可以同步到 MySQL、Elasticsearch 和 HBase 等存储中去。
- canal.admin：为 canal 提供整体配置管理、节点运维等面向运维的功能，提供相对友好的 WebUI 操作界面，方便更多用户快速和安全的操作（非必须）。

### 3.3 MySQL 配置

- 首先，修改配置文件，关键配置如下（主要是开启 binlog 写入功能，模式为 ROW，其他参数的配置以实际为准）：

```
[client]
port = 3306

[mysqld]
port = 3306
default-time-zone = '+8:00'

server-id=1
log-bin=mysql-bin
binlog_format=ROW
expire_logs_days=7
```

- 接着，需要创建一个拥有从库权限的账号，用于订阅 binlog，这里创建的账号为 `canal:canal`：

```sql
CREATE USER canal IDENTIFIED BY 'canal';  
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
-- GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' ;
FLUSH PRIVILEGES;
```

- 最后，在数据库 `cxy35` 中创建和表 `t_user`，如下：

```sql
CREATE TABLE `t_user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `username` varchar(255) DEFAULT NULL COMMENT '用户名',
  `password` varchar(255) DEFAULT NULL COMMENT '密码',
  `enabled` tinyint(1) DEFAULT '1' COMMENT '是否启用：1 启用；0 未启用',
  `locked` tinyint(1) DEFAULT '0' COMMENT '是否锁定：1 锁定；0 未锁定',
  `address` varchar(255) DEFAULT NULL COMMENT '地址',
  `nick_name` varchar(255) DEFAULT NULL COMMENT '昵称',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

### 3.4 canal.deployer 配置

- 将 `canal.deployer-1.1.5-SNAPSHOT.tar.gz` 解压到 `canal.deployer-1.1.5-SNAPSHOT`，解压完成后目录结构如下：

```
├── bin
│   ├── restart.sh
│   ├── startup.bat
│   ├── startup.sh
│   └── stop.sh
├── conf
│   ├── canal_local.properties
│   ├── canal.properties
│   └── example
│       └── instance.properties
├── lib
├── logs
│   ├── canal
│   │   └── canal.log
│   └── example
│       ├── example.log
│       └── example.log
└── plugin
```

- 修改配置文件 `conf/example/instance.properties`，主要是修改数据库相关配置：

```properties
# position info
canal.instance.master.address=127.0.0.1:3306
canal.instance.master.journal.name=
canal.instance.master.position=
canal.instance.master.timestamp=
canal.instance.master.gtid=

# username/password
canal.instance.dbUsername=canal
canal.instance.dbPassword=canal
canal.instance.connectionCharset = UTF-8

# table regex
canal.instance.filter.regex=.*\\..*
# table black regex
canal.instance.filter.black.regex=mysql\\.slave_.*
```

- 使用 `startup.sh` 或 `startup.bat` 启动 `canal.deployer` 服务，日志信息如下：

- `logs/canal/canal.log`：查看服务日志信息。
- `logs/example/example.log`：查看 instance 日志信息

### 3.5 canal.adapter 配置

- 将 `canal.adapter-1.1.5-SNAPSHOT.tar.gz` 解压到 `canal.adapter-1.1.5-SNAPSHOT`，解压完成后目录结构如下：

```
├── bin
│   ├── adapter.pid
│   ├── restart.sh
│   ├── startup.bat
│   ├── startup.sh
│   └── stop.sh
├── conf
│   ├── application.yml
│   ├── es6
│   ├── es7
│   │   ├── biz_order.yml
│   │   ├── customer.yml
│   │   └── product.yml
│   ├── hbase
│   ├── kudu
│   ├── logback.xml
│   ├── META-INF
│   │   └── spring.factories
│   └── rdb
├── lib
├── logs
│   └── adapter
│       └── adapter.log
└── plugin
```

- 修改配置文件 `conf/application.properties`，主要是修改 canal.deployer(consumer) 配置、数据源配置和客户端适配器配置（同步到 ES）：

```properties
canal.conf:
  mode: tcp #tcp #kafka rocketMQ rabbitMQ
  flatMessage: true
  zookeeperHosts:
  syncBatchSize: 1000
  retries: 0
  timeout:
  accessKey:
  secretKey:
  consumerProperties:
    # canal tcp consumer
    canal.tcp.server.host: 127.0.0.1:11111
    canal.tcp.zookeeper.hosts:
    canal.tcp.batch.size: 500
    canal.tcp.username:
    canal.tcp.password:

  srcDataSources:
    defaultDS:
      url: jdbc:mysql://127.0.0.1:3306/cxy35?useUnicode=true
      username: canal
      password: canal
  canalAdapters:
  - instance: example # canal instance Name or mq topic name
    groups:
    - groupId: g1
      outerAdapters:
      - name: logger
      - name: es7
        hosts: 127.0.0.1:9200
        properties:
          mode: rest
          security.auth: elastic:000000
          cluster.name: elasticsearch
```

- 添加配置文件 `canal-adapter/conf/es7/t_user.yml`（adapter 将会自动加载 conf/es7 下的所有 .yml 结尾的配置文件），用于配置 MySQL 中的表与 Elasticsearch 中索引的映射关系：

```yml
dataSourceKey: defaultDS
destination: example
groupId: g1
esMapping:
  _index: canal_user
  _id: _id
#  upsert: true
#  pk: id
  sql: "select u.id as _id, u.id, u.username, u.password, u.enabled, u.locked,
       u.address, u.nick_name, u.create_time, u.update_time
       from t_user u"
#  objFields:
#    _labels: array:;
  etlCondition: "where u.create_time>={}"
  commitBatch: 3000
```

- 使用 `startup.sh` 或 `startup.bat` 启动 `canal.deployer` 服务，可通过 `logs/adapter/adapter.log` 查看服务日志信息。

- 更多同步到 ES 的配置参考：[https://github.com/alibaba/canal/wiki/Sync-ES](https://github.com/alibaba/canal/wiki/Sync-ES)

### 3.6 测试

- 首先，在 Elasticsearch 中创建上述配置文件 `t_user.yml` 中对应的索引 `canal_user`，和 MySQL 中的 `t_user` 表相对应，直接在 Kibana 的 Dev Tools 中使用如下命令创建即可：

```
PUT /canal_user
{
  "mappings": {
    "properties": {
      "address": {
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart"
      },
      "create_time": {
        "type": "date"
      },
      "enabled": {
        "type": "boolean"
      },
      "id": {
        "type": "long"
      },
      "locked": {
        "type": "boolean"
      },
      "nick_name": {
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart"
      },
      "password": {
        "type": "keyword"
      },
      "update_time": {
        "type": "date"
      },
      "username": {
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart"
      }
    }
  }
}
```

创建完成后可以通过 `GET /canal_user/_mapping` 命令查看索引的结构。 

- 接着，在 MySQL 中针对 `t_user` 表做增删改操作，脚本如下：

```sql
INSERT INTO t_user(username, password, enabled, locked, address, nick_name, create_time, update_time) VALUES ('zhangsan', '123456', '1', '0', '杭州', '张三', '2020-11-13 11:05:04', '2020-11-13 11:05:04');
INSERT INTO t_user(username, password, enabled, locked, address, nick_name, create_time, update_time) VALUES ('lisi', '123456', '1', '0', '杭州', '李四', '2020-11-13 11:05:04', '2020-11-13 11:05:04');
INSERT INTO t_user(username, password, enabled, locked, address, nick_name, create_time, update_time) VALUES ('wangwu', '123456', '1', '0', '宁波', '王五', '2020-11-13 11:05:04', '2020-11-13 11:05:04');

INSERT INTO t_user(username, password, enabled, locked, address, nick_name, create_time, update_time) VALUES ('test', '123456', '1', '0', '宁波', '测试', '2020-11-13 11:05:04', '2020-11-13 11:05:04');

UPDATE t_user set nick_name='测试222' WHERE id=4;

DELETE FROM t_user WHERE id=4;
```

- 最后，通过 `GET /canal_user/_search` 命令，查看 Elasticsearch 中的数据，发现会实时同步。

### 3.7 adapter 管理 REST 接口

- 查询所有订阅同步的 canal instance 或 MQ topic：`curl http://127.0.0.1:8081/destinations`
- 数据同步开关状态：`curl http://127.0.0.1:8081/syncSwitch/example`
- 数据同步开关：`curl http://127.0.0.1:8081/syncSwitch/example/off -X PUT`
- **手动 ETL，可用于数据全量/增量同步**（如果 `params` 参数为空则全表导入, 参数对应的查询条件在配置中的 `etlCondition` 指定）：`curl http://127.0.0.1:8081/etl/es7/t_user.yml -X POST -d "params=2020-10-20 00:00:00"`
- 查看相关库总数据：`curl http://127.0.0.1:8081/count/es7/t_user.yml`

## 4 使用 canal.admin

### 4.1 简介

canal 1.1.4 版本之后，提供了 canal.admin。它为 canal 提供整体配置管理、节点运维等面向运维的功能，提供相对友好的 WebUI 操作界面，方便更多用户快速和安全的操作。

canal.admin 的核心模型主要有：

- instance：对应 canal.deployer（canal.server） 里的 instance，一个最小的订阅 MySQL 的队列。
- server：对应 canal.deployer（canal.server），一个 server 里可以包含多个 instance。
- 集群：对应一组 canal.deployer（canal.server），组合在一起面向高可用 HA 的运维。

简单解释：

- instance 因为是最原始的业务订阅诉求，它会和 server/集群 这两个面向资源服务属性的进行关联，比如 instance A 绑定到 server A 上或者集群 A 上。
- 有了任务和资源的绑定关系后，对应的资源服务就会接收到这个任务配置，在对应的资源上动态加载 instance，并提供服务。动态加载的过程，有点类似于之前的 autoScan 机制，只不过基于 canal.admin 之后可就以变为远程的 web 操作，而不需要在机器上运维配置文件。
- 将 server 抽象成资源之后，原本 `canal.deployer` 运行所需要的 `canal.properties/instance.properties` 配置文件就需要在 WebUI 上进行统一运维，每个 server 只需要以最基本的启动配置（比如知道一下 canal.admin 的 manager 地址，以及访问配置的账号、密码即可）。

### 4.2 canal.admin 配置

- 将 `canal.admin-1.1.5-SNAPSHOT.tar.gz` 解压到 `canal.admin-1.1.5-SNAPSHOT`，解压完成后目录结构如下：

```
├── bin
│   ├── restart.sh
│   ├── startup.bat
│   ├── startup.sh
│   └── stop.sh
├── conf
│   ├── application.yml
│   ├── canal_manager.sql
│   ├── canal-template.properties
│   ├── instance-template.properties
│   ├── logback.xml
│   └── public
│       ├── avatar.gif
│       ├── index.html
│       ├── logo.png
│       └── static
├── lib
└── logs
```

- 创建 canal.admin 需要使用的数据库 `canal_manager`，创建 SQL 脚本为 `conf/canal_manager.sql`，会创建如下表：

![](https://oscimg.oschina.net/oscnet/up-9a0a516f1dfe396080a5cab5c9c308f7e27.png)

---

- 修改 `canal.admin` 中的配置文件 `conf/application.yml`，主要是修改数据源配置和 canal.admin 的管理账号配置，注意需要用一个有读写权限的数据库账号（比如 root）：

```yml
server:
  port: 8089
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8

spring.datasource:
  address: 127.0.0.1:3306
  database: canal_manager
  username: root
  password: '000000'
  driver-class-name: com.mysql.jdbc.Driver
  url: jdbc:mysql://${spring.datasource.address}/${spring.datasource.database}?useUnicode=true&characterEncoding=UTF-8&useSSL=false
  hikari:
    maximum-pool-size: 30
    minimum-idle: 1

canal:
  adminUser: admin
  adminPasswd: admin
```

- 使用 `startup.sh` 或 `startup.bat` 启动 `canal.admin` 服务，可通过 `logs/admin.log` 查看服务日志信息。

### 4.3 canal.deployer 配置

- 修改 `canal.deployer` 中的配置文件 `conf/canal.properties`，将 `conf/canal_local.properties` 中的内容替换到 `conf/canal.properties` 中，主要是修改 `canal.admin` 的配置：

```properties
# register ip
canal.register.ip =

# canal admin config
canal.admin.manager = 127.0.0.1:8089
canal.admin.port = 11110
canal.admin.user = admin
# 账号密码要与 canal.admin 配置文件中的对应
canal.admin.user = admin
# 使用 MySQL 中 password 方法加密（明文：admin）
canal.admin.passwd = 4ACFE3202A5FF5CF467898FC58AAB1D615029441
# admin auto register
canal.admin.register.auto = true
canal.admin.register.cluster = 
```

- 使用 `startup.sh` 或 `startup.bat` 重新启动 `canal.deployer` 服务。

> 关于配置文件，也可以不修改 `conf/canal.properties` ，而直接使用 `conf/canal_local.properties`，此时启动时需要指定配置文件，如下：`sh bin/startup.sh local`。

### 4.4 测试

- 访问 [http://127.0.0.1:8089](http://127.0.0.1:8089) 就可以看到 canal.admin 的 Web 界面了，默认的账号密码为：`admin/123456`（对应 canal_manager.canal_user 表）。

![](https://oscimg.oschina.net/oscnet/up-497438c1a4d1ccbd728537207d9de0fdffa.png)

![](https://oscimg.oschina.net/oscnet/up-1166820ce79475cb36889236d898c00da7b.png)

- 更多 canal.admin 的配置和操作参考：[https://github.com/alibaba/canal/wiki/Canal-Admin-Guide](https://github.com/alibaba/canal/wiki/Canal-Admin-Guide)

## 5 扩展阅读

- canal 官方项目：[https://github.com/alibaba/canal](https://github.com/alibaba/canal)
- canal 官方文档：[https://github.com/alibaba/canal/wiki](https://github.com/alibaba/canal/wiki)

---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)