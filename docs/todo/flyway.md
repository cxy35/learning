Flyway 实现数据库版本管理

【TODO-待整理】Flyway 是独立于数据库的应用、管理并跟踪数据库变更的数据库版本管理工具。用通俗的话讲， Flyway 可以像 Git 那样管理不同人的代码那样，管理不同人的 sql 脚本，从而做到数据库同步。
<!-- more -->

https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247487891&idx=1&sn=a1a42fe62fd6802e14bf8bbbe131590e&chksm=e9c343f3deb4cae57eef96117cd01dd0171c9866eeac8ae2377e487413e7f4f7beccf8dcdfc2&mpshare=1&scene=1&srcid=&sharer_sharetime=1582088582563&sharer_shareid=d595861b3802327046b89c2525b17a3c&key=26c378d410f2b0d493bb0133eb0717bcef304ef0bc294c336e873e35a3c95a6668049a7ac40caf9a2d7e29a5d01d8f0abf1a65aad978cbff382da970929a439a2c9f683fef4d465e679353eab1877662&ascene=1&uin=MjAyMjk5NTg2MA%3D%3D&devicetype=Windows+7&version=62080079&lang=zh_CN&exportkey=ARUuSRpwGY1q2qB%2ByBnH%2FPA%3D&pass_ticket=O80RWBcytoKtWbB0%2BKac1%2FFrOgA7iZP1Yz%2F7H6N7IdN1CDMcz%2BPbUpJlQemqfznP

https://www.baidu.com/s?ie=UTF-8&wd=Flyway

## 1 工作模式

Flyway 不限定脚本里面的内容，但是对脚本文件的名称有一定的要求：

![](https://oscimg.oschina.net/oscnet/up-acc931ba27a6ad81e7ae63679242a1893c3.png)

版本号可以使用小版本，如 V1.1 ，具体要求：

- 版本号和版本描述之间，使用两个下划线分隔。
- 版本描述之间，使用一个下划线分隔单词。
- 版本号唯一：不允许多个脚本文件有相同的版本号。

使用Flyway升级，flyway会自动创建一张历史记录表：flyway_schema_history。这张表记录了每一次升级的记录，包括已经执行了哪些脚本，脚本的文件名，内容校验和，执行的时间和结果：

flyway在升级数据库的时候，会检查已经执行过的版本对应的脚本是否发生变化，包括脚本文件名，以及脚本内容。如果flyway检测到发生了变化，则抛出错误，并终止升级。

如果已经执行过的脚本没有发生变化，flyway会跳过这些脚本，依次执行后续版本的脚本，并在记录表中插入对应的升级记录。

所以，flyway总是幂等的，而且可以支持跨版本的升级。

如果你好奇，flyway如何检查脚本文件的内容是否有修改。你可以注意以下记录表中有一个字段checksum，它记录了脚本文件的校验和。flyway通过比对文件的校验和来检测文件的内容是否变更。

使用上面的方式，升级一个空的数据库，或者在一直使用flyway升级方案的数据库上进行升级，都不会又问题。但是，如果在已有的数据库引入flyway，就需要一些额外的工作。

flyway检测数据库中是否有历史记录表，没有则代表是第一次升级。此时，flyway要求数据库是空的，并拒绝对数据库进行升级。

你可以设置baseline-on-migrate参数为true，flyway会自动将当前的数据库记录为V1版本，然后执行升级脚本。这也表示用户所准备的脚本中，V1版本的脚本会被跳过，只有V1之后的版本才会被执行。

## 2 使用方式

### 2.1 命令行

基于命令行模式，用户可以在官网下载适合自己平台的工具包，进行相关配置之后，就可以通过命令行的方式使用 Flyway 。

### 2.2 Maven 或 Gradle

基于 Maven 或 Gradle ，用户可以通过配置插件，运行 mvn 或 gradle 命令来使用其功能。

以 Maven 为例，在 pom 文件中进行必要的配置，包括插件及插件所需要的一些数据库连接信息，就可以通过运行插件来使用其功能。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project ...>
    <properties>
        <flyway.user>postgres</flyway.user>
        <flyway.password>postgres</flyway.password>
        <flyway.url>jdbc:postgresql://localhost:5432/test?currentSchema=demo_flyway</flyway.url>
        <flyway.driver>org.postgresql.Driver</flyway.driver>
    </properties>

    <dependencies>
        ...
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        ...
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.flywaydb</groupId>
                <artifactId>flyway-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

使用 maven 命令执行插件，默认在 classpath:/db/migration 目录搜索脚本，如果该目录不存在，命令将被忽略。命令格式为：`mvn flyway:{flyway-command}`，支持如下的命令：

- `mvn flyway:migrate`：会搜索默认的脚本目录，检测并根据结果选择执行升级脚本。
- `mvn flyway:clean`：会清除指定 schema 下所有的对象，包括 table、view、triggers...，让 schema 变成空的状态。
- `mvn flyway:info`：显示指定 schema 的升级状态，当前的数据库的版本信息。
- `mvn flyway:validate`：用于校验，范围包括已升级的脚本是否改名，已升级的脚本内容是否修改。所有针对已升级的脚本进行的改动都会导致校验失败。执行 migrate 会自动进行校验，如果失败将不会做任何的 migrate 。 flyway 希望用户提供的脚本是稳定的，以免造成额外的复杂性和混乱。
- `mvn flyway:baseline`：如果用户从一个已有的数据库导出脚本，作为 flyway 的升级脚本。已存在的数据库是不需要升级的。 baseline 用于将当前数据库标记为 baseline ，并记录 version 为 1 。这表示用户继续执行 migrate 命令时，会自动跳过 V1 版本对应的脚本。而对于空的数据库，因为没有执行 baseline ，所以可以正常的执行 V1 版本对应的脚本。另外，手动修改 flyway 自动生成的 baseline 记录，将版本号改为其他的版本号，将自动跳过该版本及更早的版本。

### 2.3 Java API

基于 Java API，用户可以将 Flyway 提供的第三方包加入 classpath ，通过 Flyway 提供的 API 来使用其功能。

Flyway提供了基于Java的API包，用户可以将API包引入maven依赖，直接通过调用其API来执行相关命令。

Spring-Boot集成了Flyway，只要把API包加入classpath，spring-boot在启动应用时会去指定的目录查找脚本文件，并根据一定的策略选择或忽略执行。

使用Spring-Boot，用户不必再显式的编写代码调用API，只需要将脚本文件放在约定的目录，或者告诉Spring-Boot你把脚本文件放在哪里了。

如果用户需要实现非常灵活的迁移，Spring-Boot默认的方案无法满足，也可以尝试寻找自己编码调用API的方案。

- 在maven中引入flyway依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project ...>
    ...
    <dependencies>
        <dependency>
            <groupId>org.flywaydb</groupId>
            <artifactId>flyway-core</artifactId>
        </dependency>

        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
              ...
    </dependencies>
    ...
</project>
```

flyway-core即为我们所说的API包，除此之外，还要引入postgresql驱动包和spring-boot-starter-jdbc。

- 配置application

按照常规的方式，在application.yml文件中配置spring.datasource系列：

```xml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/test?currentSchema=demo_flyway
    driver-class-name: org.postgresql.Driver
    username: postgres
    password: postgres
```

spring为flyway准备了专属的数据源配置，但是在默认的情况下，可以直接使用spring.datasource的配置。

用户可以将脚本放在约定的位置：classpath:/db/migration，或者配置一个自定义的位置：

- 在指定的目录编写脚本

如果用户没有特地设置脚本的位置，则应该在/db/migration创建脚本。否则，在对应的位置创建脚本。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.top](https://cxy35.top)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)