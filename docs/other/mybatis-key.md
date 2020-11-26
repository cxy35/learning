---
title: MyBatis 主键回填
date: 2019-02-16 16:40:46
categories: MyBatis
tags: [MyBatis]
toc: true
---
两种方式实现 MyBatis 主键回填。
<!-- more -->

## 方式一： useGeneratedKeys

```xml
<insert id="insert" parameterType="com.xxx.model.User" useGeneratedKeys="true" keyProperty="id">
    insert into t_user (username, phone) values (#{username}, #{phone});
</insert>
```

通过在 `insert` 节点上增加 `useGeneratedKeys` 和 `keyProperty` 属性来实现，这样通过传入一个对象执行插入操作，插入完成后，这个对象的 id 就会被自动赋值，**推荐使用这种方式**。

## 方式二： LAST_INSERT_ID

```xml
<insert id="insert" parameterType="com.xxx.model.User">
    <selectKey keyProperty="id" resultType="java.lang.Integer">
        SELECT LAST_INSERT_ID()
    </selectKey>
    insert into t_user (username, phone) values (#{username}, #{phone});
</insert>
```

通过在 `insert` 节点内增加 `selectKey` 节点来实现，`LAST_INSERT_ID()` 函数是 MySQL 自带的，可以查询刚刚插入的 id 。其实这种方式更加灵活，因为 selectKey 节点中的 SQL 可以控制在插入之前/之后执行（通过设置节点的 Order 属性为 AFTER 或者 BEFORE 可以实现）。

注意：这种方式也要设置 keyProperty 来指定将查询到的数据绑定到哪个属性上。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)