---
title: 分布式任务调度与计算中心 PowerJob
date: 2020-06-07 19:48:07
categories: PowerJob
tags: [PowerJob, 分布式, 任务调度]
toc: true
---
PowerJob 是全新一代**分布式任务调度与计算框架**，能让您轻松完成作业的调度与繁杂任务的分布式计算。
<!-- more -->

## 1 简介

### 1.1 主要特性

* 使用简单：提供前端 Web 界面，允许开发者可视化地完成调度任务的管理（增、删、改、查）、任务运行状态监控和运行日志查看等功能。
* 定时策略完善：支持 CRON 表达式、固定频率、固定延迟和 API 四种定时调度策略。
* 执行模式丰富：支持单机、广播、Map、MapReduce 四种执行模式，其中 Map/MapReduce 处理器能使开发者寥寥数行代码便获得集群分布式计算的能力。
* **DAG 工作流支持**：支持在线配置任务依赖关系，可视化得对任务进行编排，同时还支持上下游任务间的数据传递。
* 执行器支持广泛：支持 Spring Bean、内置/外置 Java 类、Shell、Python 等处理器，应用范围广。
* 运维便捷：支持**在线日志**功能，执行器产生的日志可以在前端控制台页面实时显示，降低 debug 成本，极大地提高开发效率。
* 依赖精简：最小仅依赖关系型数据库（MySQL/Oracle/MS SQLServer...），扩展依赖为 MongoDB （用于存储庞大的在线日志）。
* 高可用&高性能：调度服务器经过精心设计，一改其他调度框架基于数据库锁的策略，实现了无锁化调度。部署多个调度服务器可以同时实现高可用和性能的提升（支持无限的水平扩展）。
* 故障转移与恢复：任务执行失败后，可根据配置的重试策略完成重试，只要执行器集群有足够的计算节点，任务就能顺利完成。

### 1.2 适用场景

* 有定时执行需求的业务场景：如每天凌晨全量同步数据、生成业务报表等。
* 有需要全部机器一同执行的业务场景：如使用广播执行模式清理集群日志。
* 有需要分布式处理的业务场景：比如需要更新一大批数据，单机执行耗时非常长，可以使用 Map/MapReduce 处理器完成任务的分发，调动整个集群加速计算。
* 有需要延迟执行某些任务的业务场景：比如订单过期处理等。

### 1.3 设计目标

PowerJob 的设计目标为**企业级的分布式任务调度平台**，即成为公司内部的**任务调度中间件**。整个公司统一部署调度中心 `powerjob-server`，旗下所有业务线应用只需要依赖 `powerjob-worker` 即可接入调度中心获取任务调度与分布式计算能力。

### 1.4 在线试用

- 试用地址：[http://192.168.71.53:7700/](http://192.168.71.53:7700/)
- 应用名称：powerjob-worker-samples
- 密码：123

### 1.5 同类产品对比

||QuartZ|xxl-job|SchedulerX 2.0|**PowerJob**|
|-:|-:|-:|-:|-:|
|定时类型|CRON|CRON|CRON、固定频率、固定延迟、OpenAPI|**CRON、固定频率、固定延迟、OpenAPI**|
|任务类型|内置 Java|内置 Java、GLUE Java、Shell、Python 等脚本|内置 Java、外置 Java（FatJar）、Shell、Python 等脚本|**内置 Java、外置 Java（容器）、Shell、Python 等脚本**|
|分布式任务|无|静态分片|MapReduce 动态分片|**MapReduce 动态分片**|
|在线任务治理|不支持|支持|支持|**支持**|
|日志白屏化|不支持|支持|不支持|**支持**|
|调度方式及性能|基于数据库锁，有性能瓶颈|基于数据库锁，有性能瓶颈|不详|**无锁化设计，性能强劲无上限**|
|报警监控|无|邮件|短信|**邮件和钉钉（另外开发者可基于接口方便扩展，如：短信）**|
|系统依赖|JDBC 支持的关系型数据库（MySQL、Oracle...）|MySQL|人民币|**任意 Spring Data Jpa 支持的关系型数据库（MySQL、Oracle...）** |
|DAG 工作流|不支持|不支持|支持|**支持**|

## 2 基本概念

### 2.1 分组概念

- appName：应用名称，建议与用户实际接入 PowerJob 的应用名称保持一致，**用于业务分组与隔离，一个 appName 等于一个业务集群，也就是实际的一个 Java 项目**。

### 2.2 核心概念

- 任务（Job）：描述了需要被 PowerJob 调度的任务信息，包括任务名称、调度时间、处理器信息等。
- 任务实例（JobInstance，简称 Instance）：任务（Job）被调度执行后会生成任务实例（Instance），任务实例记录了任务的运行时信息（任务与任务实例的关系类似于类与对象的关系）。
- 作业（Task）：任务实例的执行单元，一个 JobInstance 存在至少一个 Task，具体规则如下：
- 单机任务（STANDALONE）：一个 JobInstance 对应一个 Task。
- 广播任务（BROADCAST）：一个 JobInstance 对应 N 个 Task，N 为集群机器数量，即每一台机器都会生成一个 Task。
- Map/MapReduce 任务：一个 JobInstance 对应若干个 Task，由开发者手动 map 产生。
- 工作流（Workflow）：由 DAG（有向无环图）描述的一组任务（Job），用于任务编排。
- 工作流实例（WorkflowInstance）：工作流被调度执行后会生成工作流实例，记录了工作流的运行时信息。

### 2.3 扩展概念

- 容器：以 Maven 工程项目的维度组织一堆 Java 文件（开发者开发的众多 Java 处理器），**可以通过前端网页动态发布并被执行器加载，具有极强的扩展能力和灵活性**。
- OpenAPI：允许开发者通过接口来完成手工的操作，让系统整体变得更加灵活。开发者可以基于 API 便捷地扩展 PowerJob 原有的功能。

### 2.4 定时任务类型

- API：该任务只会由 powerjob-client 中提供的 OpenAPI 接口触发，server 不会主动调度。
- CRON：该任务的调度时间由 CRON 表达式指定。
- 固定频率：秒级任务，每隔多少毫秒运行一次，功能与 java.util.concurrent.ScheduledExecutorService#scheduleAtFixedRate 相同。
- 固定延迟：秒级任务，延迟多少毫秒运行一次，功能与 java.util.concurrent.ScheduledExecutorService#scheduleWithFixedDelay 相同。
- 工作流：该任务只会由其所属的工作流调度执行，server 不会主动调度该任务。如果该任务不属于任何一个工作流，该任务就不会被调度。

> 备注：固定延迟和固定频率任务统称秒级任务，这两种任务无法被停止，**只有任务被关闭或删除时才能真正停止任务**。

## 3 项目地址

- PowerJob 主项目：https://github.com/KFCFans/PowerJob
- PowerJob 前端项目：https://github.com/PowerJob/PowerJob-Console
- PowerJob 官网项目：https://github.com/PowerJob/Official-Website

## 4 项目结构说明

本项目由主体项目（`PowerJob`）和前端项目（`PowerJob-Console`）构成，其中 PowerJob 各模块说明如下：

```
├── LICENSE
├── powerjob-client // powerjob-client，普通 Jar 包，提供 OpenAPI
├── powerjob-common // 各组件的公共依赖，开发者无需感知
├── powerjob-server // powerjob-server，基于 SpringBoot 实现的调度服务器
├── powerjob-worker // powerjob-worker, 普通 Jar 包，接入 powerjob-server 的应用需要依赖该 Jar 包
├── powerjob-worker-agent // powerjob-agent，可执行 Jar 文件，可直接接入 powerjob-server 的代理应用
├── powerjob-worker-samples // 教程项目，包含了各种 Java 处理器的编写样例
├── others
└── pom.xml
```

## 5 交流与讨论

- QQ 群：
- 微信群：

---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)