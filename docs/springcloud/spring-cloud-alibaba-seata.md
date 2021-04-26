学习在 Spring Cloud 中使用 Seata 解决分布式事务问题。Seata 是 Alibaba 开源的一款分布式事务解决方案，致力于提供高性能和简单易用的分布式事务服务，本文将通过一个简单的下单业务场景来对其用法进行详细介绍。
<!-- more -->

## 1 什么是分布式事务问题？

### 1.1 单体应用

单体应用中，一个业务操作需要调用三个模块完成，此时数据的一致性由本地事务来保证。

![](https://oscimg.oschina.net/oscnet/up-6c5fee8e4e0e48c2f39c5d489142c159762.png)

### 1.2 微服务应用

随着业务需求的变化，单体应用被拆分成微服务应用，原来的三个模块被拆分成三个独立的应用，分别使用独立的数据源，业务操作需要调用三个服务来完成。此时每个服务内部的数据一致性由本地事务来保证，但是全局的数据一致性问题没法保证。

![](https://oscimg.oschina.net/oscnet/up-ce5e468a54bd2309724f7387a59ccc4202b.png)

### 1.3 小结

在微服务架构中由于全局数据一致性没法保证产生的问题就是分布式事务问题。简单来说，一次业务操作需要操作多个数据源或需要进行远程调用，就会产生分布式事务问题。

## 2 Seata 简介

Seata 是一款开源的分布式事务解决方案，致力于提供高性能和简单易用的分布式事务服务。Seata 将为用户提供了 AT、TCC、SAGA 和 XA 事务模式，为用户打造一站式的分布式解决方案。

## 3 Seata 原理和设计

### 3.1 定义一个分布式事务

我们可以把一个分布式事务理解成一个包含了若干分支事务的全局事务，全局事务的职责是协调其下管辖的分支事务达成一致，要么一起成功提交，要么一起失败回滚。此外，通常分支事务本身就是一个满足ACID的本地事务。这是我们对分布式事务结构的基本认识，与 XA 是一致的。

![](https://oscimg.oschina.net/oscnet/up-f9386de40e5877b2049d66304b332371d34.png)

### 3.2 协议分布式事务处理过程的三个组件

- Transaction Coordinator (TC)： 事务协调器，维护全局事务的运行状态，负责协调并驱动全局事务的提交或回滚；
- Transaction Manager (TM)： 控制全局事务的边界，负责开启一个全局事务，并最终发起全局提交或全局回滚的决议；
- Resource Manager (RM)： 控制分支事务，负责分支注册、状态汇报，并接收事务协调器的指令，驱动分支（本地）事务的提交和回滚。

![](https://oscimg.oschina.net/oscnet/up-cefb3762865e90d5d6daba6440dff719804.png)

### 3.3 一个典型的分布式事务过程

- TM 向 TC 申请开启一个全局事务，全局事务创建成功并生成一个全局唯一的 XID；
- XID 在微服务调用链路的上下文中传播；
- RM 向 TC 注册分支事务，将其纳入 XID 对应全局事务的管辖；
- TM 向 TC 发起针对 XID 的全局提交或回滚决议；
- TC 调度 XID 下管辖的全部分支事务完成提交或回滚请求。

![](https://oscimg.oschina.net/oscnet/up-7cb902dfec632d17594f2458fbe7edaecb5.png)

上述案例解决方案：

![](https://oscimg.oschina.net/oscnet/up-78d36fc0ae795139acf322e71e92bbdb6f8.png)

## 4 Seata 使用

### 4.1 seata-server 安装与配置

- 首先，从官网下载 `seata-server`，下载地址：[https://github.com/seata/seata/releases](https://github.com/seata/seata/releases)，这里下载的是 `seata-server-1.4.1.zip`，解压到需要目录中。

- 在 `conf/file.conf` 文件中指定事务日志存储的模式（默认为 `file`），这里我们修改为 `db` 模式，需要提前创建数据库 `seata`（建表语句在这里下载：[https://github.com/seata/seata/tree/develop/script/server/db](https://github.com/seata/seata/tree/develop/script/server/db)），之后配置数据库连接信息。

```bash
## transaction log store, only used in seata-server
store {
  ## store mode: file、db、redis
  mode = "db" # 修改为 db

  ## ...

  ## database store property
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp)/HikariDataSource(hikari) etc.
    datasource = "druid"
    ## mysql/oracle/postgresql/h2/oceanbase etc.
    dbType = "mysql"
    driverClassName = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://127.0.0.1:3306/seata"
    user = "root"
    password = "000000"
    minConn = 5
    maxConn = 100
    globalTable = "global_table"
    branchTable = "branch_table"
    lockTable = "lock_table"
    queryLimit = 100
    maxWait = 5000
  }

  ## ...

}
```

- 在 `conf/registry.conf` 文件中指定注册中心和配置中心的类型（默认为 `file`），这里我们修注册中心类型为 `nacos`，并修改 `nacos` 连接信息。

```bash
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "nacos" # 修改为 nacos
  loadBalance = "RandomLoadBalance"
  loadBalanceVirtualNodes = 10

  nacos {
    application = "seata-server"
    serverAddr = "127.0.0.1:8848"
    group = "SEATA_GROUP"
    namespace = ""
    cluster = "default"
    username = "nacos"
    password = "nacos"
  }

  # ...
}
```

- 先启动 Nacos，再使用 `bin/seata-server.bat` 文件启动 seata-server。

### 4.2 seata-client 使用

#### 4.2.1 创建业务数据库和表

- 创建存储订单的数据表 `seata-order.order` ：

```sql
CREATE TABLE `order` (
  `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `user_id` bigint(11) DEFAULT NULL COMMENT '用户id',
  `product_id` bigint(11) DEFAULT NULL COMMENT '产品id',
  `count` int(11) DEFAULT NULL COMMENT '数量',
  `money` decimal(11,0) DEFAULT NULL COMMENT '金额',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8;

ALTER TABLE `order` ADD COLUMN `status` int(1) DEFAULT NULL COMMENT '订单状态：0：创建中；1：已完结' AFTER `money` ;
```

- 创建存储库存的数据表 `seata-storage.storage`：

```sql
CREATE TABLE `storage` (
                         `id` bigint(11) NOT NULL AUTO_INCREMENT,
                         `product_id` bigint(11) DEFAULT NULL COMMENT '产品id',
                         `total` int(11) DEFAULT NULL COMMENT '总库存',
                         `used` int(11) DEFAULT NULL COMMENT '已用库存',
                         `residue` int(11) DEFAULT NULL COMMENT '剩余库存',
                         PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;

INSERT INTO `seata-storage`.`storage` (`id`, `product_id`, `total`, `used`, `residue`) VALUES ('1', '1', '100', '0', '100');
```

- 创建存储账户信息的数据表 `seata-account.account`：

```sql
CREATE TABLE `account` (
  `id` bigint(11) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `user_id` bigint(11) DEFAULT NULL COMMENT '用户id',
  `total` decimal(10,0) DEFAULT NULL COMMENT '总额度',
  `used` decimal(10,0) DEFAULT NULL COMMENT '已用余额',
  `residue` decimal(10,0) DEFAULT '0' COMMENT '剩余可用额度',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;

INSERT INTO `seata-account`.`account` (`id`, `user_id`, `total`, `used`, `residue`) VALUES ('1', '1', '1000', '0', '1000');
```

- 在上述业务数据库中分别创建日志回滚表 `undo_log` ，建表语句在这里下载：[https://github.com/seata/seata/tree/develop/script/client](https://github.com/seata/seata/tree/develop/script/client)。

- 完整数据库示意图如下：

![](https://oscimg.oschina.net/oscnet/up-c87cc550c7b115ba10be7278ba3d8cc16f3.png)

#### 4.2.2 创建三个服务并完成配置

- 订单服务 `seata-order-service`。
- 库存服务 `seata-storage-service`。
- 账户服务 `seata-account-service`。

当用户下单时，会在订单服务中创建一个订单，然后通过远程调用库存服务来扣减下单商品的库存，再通过远程调用账户服务来扣减用户账户里面的余额，最后在订单服务中修改订单状态为已完成。该操作跨越三个数据库，有两次远程调用，很明显会有分布式事务问题。

- 首先，对三个服务分别进行客户端配置，下面以订单服务 `seata-order-service` 为例（其他服务类似）。

从 [https://github.com/seata/seata/tree/develop/script/client/conf](https://github.com/seata/seata/tree/develop/script/client/conf) 中获取配置文件 `file.conf` 和 `registry.conf` ，拷贝到 `src/main/resources` 中。

修改 `file.conf`，主要是修改自定义事务组名称：

```bash
service {
  #transaction service group mapping
  vgroupMapping.my_test_tx_group = "default" # 修改自定义事务组名称为 my_test_tx_group
  #only support when registry.type=file, please don't set multiple addresses
  default.grouplist = "127.0.0.1:8091"
  #degrade, current not support
  enableDegrade = false
  #disable seata
  disableGlobalTransaction = false
}
```

修改 `registry.conf` ，主要是将注册中心改为 nacos：

```bash
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa、custom
  type = "nacos" # 修改为 nacos

  nacos {
    application = "seata-server"
    serverAddr = "127.0.0.1:8848"
    group = "SEATA_GROUP"
    namespace = ""
    cluster = "default"
    username = "nacos"
    password = "nacos"
  }

  # ...
}
```

- 接着，对三个服务分别进行代码编写和配置，下面以订单服务 `seata-order-service` 为例（其他服务类似）。

修改配置文件 `application.yml`：

```yaml
server:
  port: 8180
spring:
  application:
    name: seata-order-service # 自定义事务组名称需要与上文一致
  cloud:
    alibaba:
      seata:
        tx-service-group: my_test_tx_group
    nacos:
      discovery:
        server-addr: localhost:8848
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/seata-order
    username: root
    password: '000000'
feign:
  hystrix:
    enabled: false
logging:
  level:
    io:
      seata: info
mybatis:
  mapperLocations: classpath:mapper/*.xml
```

在启动类中取消数据源的自动创建：

```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
@EnableDiscoveryClient
@EnableFeignClients
public class SeataOrderServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(SeataOrderServiceApplication.class, args);
    }

}
```

创建配置类，使用 Seata 对数据源进行代理：

```java
/**
 * 使用Seata对数据源进行代理
 */
@Configuration
public class DataSourceProxyConfig {

    @Value("${mybatis.mapperLocations}")
    private String mapperLocations;

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource druidDataSource(){
        return new DruidDataSource();
    }

    @Bean
    public DataSourceProxy dataSourceProxy(DataSource dataSource) {
        return new DataSourceProxy(dataSource);
    }

    @Bean
    public SqlSessionFactory sqlSessionFactoryBean(DataSourceProxy dataSourceProxy) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSourceProxy);
        sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources(mapperLocations));
        sqlSessionFactoryBean.setTransactionFactory(new SpringManagedTransactionFactory());
        return sqlSessionFactoryBean.getObject();
    }

}
```

使用 `@GlobalTransactional` 注解开启分布式事务：

```java
/**
 * 订单业务实现类
 */
@Service
public class OrderServiceImpl implements OrderService {

    private static final Logger LOGGER = LoggerFactory.getLogger(OrderServiceImpl.class);

    @Autowired
    private OrderDao orderDao;
    @Autowired
    private StorageService storageService;
    @Autowired
    private AccountService accountService;

    /**
     * 创建订单->调用库存服务扣减库存->调用账户服务扣减账户余额->修改订单状态
     */
    @Override
    @GlobalTransactional(name = "order-create",rollbackFor = Exception.class)
    public void create(Order order) {
        LOGGER.info("------->下单开始");
        //本应用创建订单
        orderDao.create(order);

        //远程调用库存服务扣减库存
        LOGGER.info("------->order-service中扣减库存开始");
        storageService.decrease(order.getProductId(),order.getCount());
        LOGGER.info("------->order-service中扣减库存结束");

        //远程调用账户服务扣减余额，对应的服务中会模拟超时异常（代码已注释）
        LOGGER.info("------->order-service中扣减余额开始");
        accountService.decrease(order.getUserId(),order.getMoney());
        LOGGER.info("------->order-service中扣减余额结束");

        //修改订单状态为已完成
        LOGGER.info("------->order-service中修改订单状态开始");
        orderDao.update(order.getUserId(),0);
        LOGGER.info("------->order-service中修改订单状态结束");

        LOGGER.info("------->下单结束");
    }
}
```

#### 4.2.3 功能演示

- 运行 seata-order-service、seata-storage-service 和 seata-account-service 三个服务。

- 数据库初始信息状态：

![](https://oscimg.oschina.net/oscnet/up-23ffdae3252c9dd887ae4333e84b38e4144.png)

- 调用接口 [http://localhost:8180/order/create?userId=1&productId=1&count=10&money=100](http://localhost:8180/order/create?userId=1&productId=1&count=10&money=100) 进行正常下单操作，之后查看数据库，数据正常变化：

![](https://oscimg.oschina.net/oscnet/up-7545a9c06d141cf8d546554d00f0608b0ff.png)

- 我们在 `seata-account-service` 中修改代码，模拟超时异常，之后再次调用下单接口：

```java
/**
 * 账户业务实现类
 */
@Service
public class AccountServiceImpl implements AccountService {

    private static final Logger LOGGER = LoggerFactory.getLogger(AccountServiceImpl.class);
    @Autowired
    private AccountDao accountDao;

    /**
     * 扣减账户余额
     */
    @Override
    public void decrease(Long userId, BigDecimal money) {
        LOGGER.info("------->account-service中扣减账户余额开始");
        //模拟超时异常，全局事务回滚
//        try {
//            Thread.sleep(30*1000);
//        } catch (InterruptedException e) {
//            e.printStackTrace();
//        }
        accountDao.decrease(userId,money);
        LOGGER.info("------->account-service中扣减账户余额结束");
    }
}
```

- 此时我们可以发现下单后数据库数据并没有任何改变，符合预期：

![](https://oscimg.oschina.net/oscnet/up-c9e884ca0f640c18f83bcdbd4ff31446ef0.png)

- 接着，我们可以在 `seata-order-service` 中注释掉 `@GlobalTransactional(name = "order-create",rollbackFor = Exception.class)` ，看看没有 Seata 的分布式事务管理会发生什么情况。由于 `seata-account-service` 的超时会导致当库存和账户金额扣减后订单状态并没有设置为已经完成，而且由于远程调用的重试机制，账户余额还会被多次扣减：

![](https://oscimg.oschina.net/oscnet/up-a0666a9b3d77d03a8e1843cb5ed852a6c44.png)

---

- [Spring Cloud 教程合集](https://mp.weixin.qq.com/s/SBmcs2bxumhNz4kky1pl-A)（微信左下方**阅读全文**可直达）。
- Spring Cloud 教程合集示例代码：[https://github.com/cxy35/spring-cloud-samples](https://github.com/cxy35/spring-cloud-samples)
- 本文示例代码：[https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-alibaba-seata](https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-alibaba-seata)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)