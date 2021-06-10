手把手带你安装 Elasticsearch。
<!-- more -->

## 1 准备工作

从 [https://www.elastic.co/cn/downloads/elasticsearch](https://www.elastic.co/cn/downloads/elasticsearch) 中下载最新稳定版本的安装文件，如：`https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.13.1-linux-x86_64.tar.gz`，历史版本可以访问：[https://www.elastic.co/downloads/past-releases](https://www.elastic.co/downloads/past-releases)。如果是在 `Linux` 下安装，可以直接使用 `wget` 命令。

![](https://oscimg.oschina.net/oscnet/up-174e727756aeeaea0beef15f93c93a2bac6.png)

另外，如果想要更快速的下载（但版本不一定是最新的），可以访问：[https://elasticsearch.cn/download/](https://elasticsearch.cn/download/)，里面包含 `Elastic` 技术栈所需所有安装文件的下载。

## 2 安装

```bash
cd /usr/local/elastic
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.13.1-linux-x86_64.tar.gz
tar -xvzf elasticsearch-7.13.1-linux-x86_64.tar.gz
cd elasticsearch-7.13.1
```

---

文件目录布局：

|类型|描述|默认位置|设置|
|:-|:-|:-|:-|
|home|Elasticsearch 主目录或 $ES_HOME|通过解压缩归档创建的目录||
|bin|二进制脚本包括用于启动节点的 elasticsearch 和用于安装插件的 elasticsearch-plugin|$ES_HOME/bin||
|conf|配置文件包括 elasticsearch.ym|$ES_HOME/config|ES_PATH_CONF|
|data|节点上分配的每个索引/分片的数据文件的位置。可以容纳多个位置。|$ES_HOME/data|path.data|
|logs|log 文件位置|$ES_HOME/logs|path.logs|
|plugins|插件文件位置。每个插件都将包含在一个子目录中。|$ES_HOME/plugins||
|repo|共享文件系统存储库位置。可以容纳多个位置。文件系统存储库可以放在此处指定的任何目录的任何子目录中。|没有配置|path.repo|
|script|脚本文件的位置|$ES_HOME/scripts|path.scripts|

默认情况下，所有文件和目录都包含在 $ES_HOME 中 - 解压缩归档时创建的目录。但是，建议更改 config 目录，数据目录和 logs 目录的默认位置，以便以后不删除重要数据。

## 3 配置

修改配置文件 `./conf/elasticsearch.yml`，常用配置如下：

```yml
#配置集群名称，默认elasticsearch
#cluster.name: my-application

#配置数据目录
#path.data: /path/to/data

#配置日志目录
#path.logs: /path/to/logs

#配置允许访问的IP地址，默认只允许本机访问
network.host: 0.0.0.0

#配置端口，默认9200
#http.port: 9200

discovery.seed_hosts: ["host1"]
```

另外，也可以在启动时配置 Elasticsearch，见下文。

## 4 启动

首先在 Linux 系统中新建 `elastic` 用户，用于启动 Elasticsearch，否则会报错：`java.lang.RuntimeException: can not run elasticsearch as root`

```bash
groupadd elastic
useradd elastic -g elastic
#passwd es

cd /usr/local/elastic
chown -R elastic:elastic elasticsearch-7.13.1

su elastic
```

```bash
cd /usr/local/elastic/elasticsearch-7.13.1

#前台运行
# ./bin/elasticsearch

#后台运行
nohup ./bin/elasticsearch &

#启动时指定 JVM 的内存大小
# ES_JAVA_OPTS="-Xms512m -Xmx512m" ./bin/elasticsearch
```

---

浏览器访问 `[http://localhost:9200](http://localhost:9200)` 验证是否启动成功。

或者通过命令验证：`curl 'http://localhost:9200'`。

```
{
  "name" : "myname",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "NnoeViFTRMy8vBEyiFHfZg",
  "version" : {
    "number" : "7.13.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "9a7758028e4ea59bcab41c12004603c5a7dd84a9",
    "build_date" : "2021-05-28T17:40:59.346932922Z",
    "build_snapshot" : false,
    "lucene_version" : "8.8.2",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

---

默认情况下，`Elasticsearch` 从 `$ES_HOME/config/elasticsearch.yml` 文件加载其配置。也可以在命令行中使用 `-E` 语法指定可在配置文件中指定的任何配置。这种情况特别适合同样一个 Elasticsearch 的安装运行多个 Elasticsearch 的实例，这样我们可以轻松建立 replica。

```bash
#启动时指定数据目录
# ./bin/elasticsearch -E path.data=/data/elasticsearch

#启动时指定 node 名字，默认为 elasticsearch
# ./bin/elasticsearch -E node.name=mynodename

#启动时同时指定多个参数
# ./bin/elasticsearch -d -E cluster.name=my_cluster -E node.name=node_1 -E http.host="localhost","mac"
```

提示：通常，应将任何群集范围的设置（如：`cluster.name`）添加到 `elasticsearch.yml` 配置文件中，而可以在命令行上指定任何特定于节点的设置（如：`node.name`）。

## 5 关闭

```bash
ps -ef|grep elasticsearch
kill -9 xxx
```

## 6 启用安全认证

修改配置文件 `./conf/elasticsearch.yml`，在最后面增加如下配置：

```yml
discovery.type: single-node

xpack.security.enabled: true
xpack.license.self_generated.type: basic
xpack.security.transport.ssl.enabled: true
```

关闭并重新启动 Elasticsearch，之后执行命令，分别为每个内置用户手动设置密码：

```bash
cd /usr/local/elastic/elasticsearch-7.13.1

./bin/elasticsearch-setup-passwords interactive
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,kibana_system,logstash_system,beats_system,remote_monitoring_user.
You will be prompted to enter passwords as the process progresses.
Please confirm that you would like to continue [y/N]y


Enter password for [elastic]: 
Reenter password for [elastic]: 
Enter password for [apm_system]: 
Reenter password for [apm_system]: 
Enter password for [kibana_system]: 
Reenter password for [kibana_system]: 
Enter password for [logstash_system]: 
Reenter password for [logstash_system]: 
Enter password for [beats_system]: 
Reenter password for [beats_system]: 
Enter password for [remote_monitoring_user]: 
Reenter password for [remote_monitoring_user]: 
Changed password for user [apm_system]
Changed password for user [kibana_system]
Changed password for user [kibana]
Changed password for user [logstash_system]
Changed password for user [beats_system]
Changed password for user [remote_monitoring_user]
Changed password for user [elastic]
```

配置之后，访问 Elasticsearch 需要带上认证信息，如：

```bash
curl -u elastic:123456 'http://localhost:9200'

# 或 curl 'http://elastic:123456@localhost:9200'
```

如果想要随机密码，可以执行：`./bin/elasticsearch-setup-passwords auto`

## 7 报错汇总

- 报错描述：`system call filters failed to install; check the logs and fix your configuration or disable system call filters at your own risk`。
- 报错原因：Centos6 不支持 SecComp，而 ES 默认 bootstrap.system_call_filter 为 true 进行检测，所以导致检测失败，失败后直接导致 ES 不能启动。
- 报错解决：修改配置文件 `./conf/elasticsearch.yml`，在 Memory 下面增加如下配置：

```yml
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
```

---

- [如何在 Linux，MacOS 及 Windows 上进行安装 Elasticsearch](https://elasticstack.blog.csdn.net/article/details/99413578)
- [Elasticsearch：设置 Elastic 账户安全](https://elasticstack.blog.csdn.net/article/details/100548174)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)