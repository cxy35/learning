---
title: IntelliJ IDEA 插件 EasyCode（代码自动生成）
date: 2020-02-18 09:37:50
categories: IDE
tags: [IDE, IntelliJ IDEA, 插件, EasyCode]
toc: true
---
IntelliJ IDEA 中的插件 EasyCode，用于**代码自动生成**，支持模板自定义、导入、导出，方便团队之间共享。
<!-- more -->

## 1 介绍

- 基于 IntelliJ IDEA 开发的代码生成插件，支持自定义任意模板（Java，html，js，xml）。
- 只要是与数据库相关的代码都可以通过自定义模板来生成。支持数据库类型与 java 类型映射关系配置。
- 支持同时生成多张表的代码。每张表有独立的配置信息。完全的个性化定义，规则由你设置。

## 2 安装

`File -> Settings -> Plugins`，在插件市场中搜索 `EasyCode` 安装，重启 IDEA。

## 3 使用

### 3.1 创建项目

`File -> New Project -> Spring Initializr` 。

### 3.2 添加数据源

EasyCode 是基于 IDEA 上的 Database Tools 开发的，因此要通过 IDEA 上的 `Database` 连接数据源。

![](https://oscimg.oschina.net/oscnet/up-74e0b0d3c2e6245e9e29f4d21661cfdc150.JPEG)

配置数据库连接信息。

![](https://oscimg.oschina.net/oscnet/up-a099792cfa639dddcd549ffef7702274818.JPEG)

### 3.3 生成代码

选择对应的数据库和表（支持多张表同时生成），右键单击，选择 `EasyCode -> Generate Code` 。

![](https://oscimg.oschina.net/oscnet/up-f4ac0513e30aadd503dcaf89507ac53d008.JPEG)

可能需要添加部分数据库类型与 Java 类型的映射关系。

![](https://oscimg.oschina.net/oscnet/up-2eb4107d45073537d99d20caff974e0add7.JPEG)

![](https://oscimg.oschina.net/oscnet/up-76f65701be38eebb9d714e2c4e12df41c8d.JPEG)

支持单张表单独配置，右键单击，选择 `EasyCode -> Config Table` 。

![](https://oscimg.oschina.net/oscnet/up-0241e081fb46cfd6894840d7acc1310cf98.JPEG)

![](https://oscimg.oschina.net/oscnet/up-c8c5f8e3b520aa77ae108af2960cd98ab3c.JPEG)

配置生成代码。

![](https://oscimg.oschina.net/oscnet/up-a5d7686239f279e43de922fac4b39372c8d.JPEG)

如果 `controller/entity/service/dao` 等包不存在会提示自动创建。最终生成的代码如下：

![](https://oscimg.oschina.net/oscnet/up-1f49b8c77616760e708e796ffa9703beb20.JPEG)

### 3.4 自定义模板

支持**自定义模板**，并且可以**实时调试**。

`File -> Settings -> Easy Code -> Template Setting` 。

![](https://oscimg.oschina.net/oscnet/up-e5bc71bd3cd7464e0e0ecd470388c5270f7.JPEG)

**建议自己新建一套模板（包括 `Type Mapper/Template Setting/Global Config` 等），默认的 Default 模板供参考**。

### 3.5 模板共享

支持模板导入、导出，方便团队之间共享。

`File -> Settings -> Easy Code` 。

![](https://oscimg.oschina.net/oscnet/up-71913d7e081d75241a66b147f77909b8cb5.png)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)